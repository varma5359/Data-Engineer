# Day 42 Project Workbook — Customer History Model (SCD Type 1, 2 and 3)

**Database:** PostgreSQL  
**Run location:** pgAdmin Query Tool or psql. Use one query session because this workbook uses TEMP tables.  
**Goal:** Build a customer history model, run three change strategies, and prove that historical lookups return the correct city.

## Business scenario

TechStore customer 501 (Asha) initially lives in Hyderabad. On 15 April 2026, she moves to Bengaluru. The reporting team asks:

1. What is Asha's current city?
2. Where did Asha live on 1 March?
3. How many city versions do we retain?
4. Which SCD approach would meet each reporting need?

For this lab, effective dates are **inclusive calendar dates**. In production timestamp models, prefer and consistently document half-open intervals (start inclusive, end exclusive).

## Step 0 — Reset this lab safely

Run only in a training database. TEMP tables disappear when your database session closes.

```sql
DROP TABLE IF EXISTS customer_type1;
DROP TABLE IF EXISTS customer_type3;
DROP TABLE IF EXISTS dim_customer_scd2;
```

Expected: three DROP messages (or notices that tables did not exist). If you use a fresh session, these tables may not exist; that is okay.

## Step 1 — Type 1: keep only the latest city

```sql
CREATE TEMP TABLE customer_type1 (
    customer_id INTEGER PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    city VARCHAR(50) NOT NULL
);

INSERT INTO customer_type1 VALUES (501, 'Asha', 'Hyderabad');

SELECT * FROM customer_type1;
```

Expected first result:

| customer_id | customer_name | city |
|---:|---|---|
| 501 | Asha | Hyderabad |

Now apply the move:

```sql
UPDATE customer_type1
SET city = 'Bengaluru'
WHERE customer_id = 501;

SELECT * FROM customer_type1;
```

Expected second result: one row, city = **Bengaluru**. Hyderabad has been overwritten in this table.

**Explain:** UPDATE changes the existing row. It does not add a history record. The WHERE clause is essential; without it, every row could be updated.

## Step 2 — Type 2: preserve every city version

Create the dimension:

```sql
CREATE TEMP TABLE dim_customer_scd2 (
    customer_key INTEGER PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    customer_name VARCHAR(100) NOT NULL,
    city VARCHAR(50) NOT NULL,
    effective_from DATE NOT NULL,
    effective_to DATE NOT NULL,
    is_current BOOLEAN NOT NULL,
    CHECK (effective_from <= effective_to)
);
```

Insert the first version:

```sql
INSERT INTO dim_customer_scd2 VALUES
(1001, 501, 'Asha', 'Hyderabad',
 DATE '2026-01-10', DATE '9999-12-31', TRUE);
```

Close the old version on the day before the move:

```sql
UPDATE dim_customer_scd2
SET effective_to = DATE '2026-04-14',
    is_current = FALSE
WHERE customer_id = 501 AND is_current = TRUE;
```

Insert the new version:

```sql
INSERT INTO dim_customer_scd2 VALUES
(1002, 501, 'Asha', 'Bengaluru',
 DATE '2026-04-15', DATE '9999-12-31', TRUE);
```

Inspect all versions:

```sql
SELECT customer_key, customer_id, city, effective_from, effective_to, is_current
FROM dim_customer_scd2
ORDER BY effective_from;
```

Expected:

| customer_key | customer_id | city | effective_from | effective_to | is_current |
|---:|---:|---|---|---|---|
| 1001 | 501 | Hyderabad | 2026-01-10 | 2026-04-14 | false |
| 1002 | 501 | Bengaluru | 2026-04-15 | 9999-12-31 | true |

**Why two keys?** customer_id identifies Asha across time. customer_key identifies one particular version. Facts should reference the version valid for their event date when historical attribute analysis is required.

## Step 3 — Ask historical questions

### 3A. Current city

```sql
SELECT customer_id, city
FROM dim_customer_scd2
WHERE customer_id = 501 AND is_current = TRUE;
```

Expected: **Bengaluru**.

### 3B. City on 1 March 2026

```sql
SELECT customer_key, city
FROM dim_customer_scd2
WHERE customer_id = 501
  AND DATE '2026-03-01' >= effective_from
  AND DATE '2026-03-01' <= effective_to;
```

Expected: key 1001, **Hyderabad**.

### 3C. City on 1 May 2026

Change the date literal to DATE '2026-05-01'. Expected: key 1002, **Bengaluru**.

**Checkpoint:** each historical lookup must return exactly one row. Zero rows means a gap/missing history; more than one means overlapping versions.

## Step 4 — Type 3: retain current and previous city only

```sql
CREATE TEMP TABLE customer_type3 (
    customer_id INTEGER PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    current_city VARCHAR(50) NOT NULL,
    previous_city VARCHAR(50)
);

INSERT INTO customer_type3 VALUES (501, 'Asha', 'Hyderabad', NULL);

UPDATE customer_type3
SET previous_city = current_city,
    current_city = 'Bengaluru'
WHERE customer_id = 501;

SELECT * FROM customer_type3;
```

Expected: current_city = Bengaluru; previous_city = Hyderabad. A further move would overwrite previous_city, so Type 3 is not a complete timeline.

## Step 5 — Validate the Type 2 model

### 5A. More than one current row per customer

```sql
SELECT customer_id, COUNT(*) AS current_count
FROM dim_customer_scd2
WHERE is_current = TRUE
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

Expected: **0 rows**.

### 5B. Current row missing

```sql
SELECT customer_id
FROM dim_customer_scd2
GROUP BY customer_id
HAVING SUM(CASE WHEN is_current THEN 1 ELSE 0 END) = 0;
```

Expected: **0 rows** for customers expected to be active.

### 5C. Date ranges overlap (inclusive-date convention)

```sql
SELECT a.customer_id, a.customer_key AS version_a, b.customer_key AS version_b
FROM dim_customer_scd2 a
JOIN dim_customer_scd2 b
  ON a.customer_id = b.customer_id
 AND a.customer_key < b.customer_key
 AND a.effective_from <= b.effective_to
 AND b.effective_from <= a.effective_to;
```

Expected: **0 rows**. If rows appear, inspect the two effective ranges; a date may belong to both versions.

### 5D. Exactly one version for each test date

```sql
SELECT COUNT(*) AS matching_versions
FROM dim_customer_scd2
WHERE customer_id = 501
  AND DATE '2026-03-01' BETWEEN effective_from AND effective_to;
```

Expected: **1**.

## Step 6 — Failure drills

For each failure, state the symptom, cause, fix and prevention.

1. The current city query returns two rows.  
   **Likely cause:** two current records. Find them with Step 5A; correct the load transaction and enforce a one-current-row rule.
2. A March order maps to Bengaluru.  
   **Likely cause:** fact lookup used the current row instead of event date. Resolve the dimension version using order date.
3. A historical date returns zero rows.  
   **Likely cause:** date gap, missing prior version or incorrect boundary convention. Inspect adjacent ranges.
4. A repeated daily file creates a new version every day.  
   **Likely cause:** loader inserts without comparing tracked attributes. Create a new version only when a tracked value changes.
5. Two updates happen on the same date.  
   **Design question:** date-only inclusive ranges cannot represent intraday order reliably. Use timestamps and a documented half-open interval, or define a business rule for same-day changes.

## Step 7 — Your submission

- [ ] Screenshot/output of Type 1 before and after.
- [ ] Type 2 history table output.
- [ ] Current-city query output.
- [ ] March and May historical lookup outputs.
- [ ] Type 3 output.
- [ ] Three validation queries with zero-row evidence.
- [ ] One-page explanation of which SCD type TechStore should use and why.
- [ ] Explain the difference between customer_id (business key) and customer_key (version/surrogate key).

## Answer key: choose the right strategy

- **Type 1:** overwrite; use for corrections/current-only attributes.
- **Type 2:** add a version; use for complete historical analysis.
- **Type 3:** current plus limited prior value; use for a small comparison requirement.

## Production note

The lab uses manual SQL and fixed surrogate keys to teach the sequence. A production loader must generate keys safely, detect changes, deduplicate source records, run close-and-insert in a transaction, handle concurrent loads, and validate history ranges. Do not copy fixed keys or dates into a real pipeline.
