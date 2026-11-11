# Day 43 Project Workbook — Warehouse Key Management

**Database:** PostgreSQL  
**Run location:** pgAdmin Query Tool or psql, in one session (TEMP tables).  
**Goal:** Prove how source identities, warehouse surrogate keys and relationships work; test duplicate and orphan protection.

## Business scenario

TechStore receives customer records from CRM and Loyalty. Both sources can assign customer ID 501. They are not automatically the same person. Orders use order ID and line number.

## Step 0 — Start a clean training session

Run in a training database only:

```sql
DROP TABLE IF EXISTS fact_order_line_key_demo;
DROP TABLE IF EXISTS dim_customer_key_demo;
```

TEMP tables disappear when the session closes.

## Step 1 — Create a customer dimension with a warehouse key

```sql
CREATE TEMP TABLE dim_customer_key_demo (
    customer_key INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    source_system VARCHAR(30) NOT NULL,
    source_customer_id VARCHAR(30) NOT NULL,
    customer_name VARCHAR(100) NOT NULL,
    UNIQUE (source_system, source_customer_id)
);
```

**Expected:** CREATE TABLE.

- Identity generates customer_key inside PostgreSQL.
- PRIMARY KEY means each warehouse key is unique and not null.
- UNIQUE(source_system, source_customer_id) prevents one source identity from being loaded twice.
- Source ID is VARCHAR to avoid assuming all identifiers are numeric.

Insert two source identities:

```sql
INSERT INTO dim_customer_key_demo
(source_system, source_customer_id, customer_name)
VALUES ('CRM', '501', 'Asha'), ('Loyalty', '501', 'Asha L');
```

**Expected:** INSERT 0 2.

Inspect:

```sql
SELECT customer_key, source_system, source_customer_id, customer_name
FROM dim_customer_key_demo ORDER BY customer_key;
```

Expected: two rows with distinct generated keys. Exact values depend on the identity sequence; do not assume they start at 1.

**Checkpoint:** Why are both rows allowed although source_customer_id is 501? Because uniqueness is scoped by source_system.

## Step 2 — Demonstrate duplicate source identity rejection

```sql
INSERT INTO dim_customer_key_demo
(source_system, source_customer_id, customer_name)
VALUES ('CRM', '501', 'Duplicate Asha');
```

Expected: unique-constraint violation; existing CRM 501 row remains unchanged.

## Step 3 — Create order-line facts with composite key and foreign key

```sql
CREATE TEMP TABLE fact_order_line_key_demo (
    order_id INTEGER NOT NULL,
    line_number INTEGER NOT NULL,
    customer_key INTEGER NOT NULL,
    amount NUMERIC(12,2) NOT NULL CHECK (amount >= 0),
    PRIMARY KEY (order_id, line_number),
    FOREIGN KEY (customer_key) REFERENCES dim_customer_key_demo(customer_key)
);
```

Expected: CREATE TABLE.

- PRIMARY KEY(order_id, line_number): each line number is unique inside an order; line numbers can repeat across orders.
- FOREIGN KEY ensures customer_key points to a dimension row.
- CHECK rejects negative amounts under this lab's sale-line rule; refunds need a separately documented model.

## Step 4 — Insert valid fact rows

Find the CRM warehouse key:

```sql
SELECT customer_key FROM dim_customer_key_demo
WHERE source_system = 'CRM' AND source_customer_id = '501';
```

Copy the returned integer into the next statement instead of the `CRM_KEY` placeholder:

```sql
INSERT INTO fact_order_line_key_demo VALUES (7001, 1, CRM_KEY, 2500.00);
```

Example: if the query returns 1, use `(7001, 1, 1, 2500.00)`. Do not assume your key is 1.

Expected: INSERT 0 1. Add a second line with the same actual key:

```sql
INSERT INTO fact_order_line_key_demo VALUES (7001, 2, CRM_KEY, 900.00);
```

Expected: INSERT 0 1; order_id repeats, line_number differs.

## Step 5 — Duplicate composite key test

```sql
INSERT INTO fact_order_line_key_demo VALUES (7001, 1, CRM_KEY, 300.00);
```

Expected: primary-key violation because (7001, 1) already exists.

## Step 6 — Foreign key test

Confirm 999999 is absent:

```sql
SELECT customer_key FROM dim_customer_key_demo WHERE customer_key = 999999;
```

Expected: zero rows. Then run:

```sql
INSERT INTO fact_order_line_key_demo VALUES (7002, 1, 999999, 100.00);
```

Expected: foreign-key violation.

## Step 7 — Orphan validation query

A foreign key prevents new orphans here. This diagnostic is useful for legacy tables or systems without enforced constraints:

```sql
SELECT f.order_id, f.line_number, f.customer_key
FROM fact_order_line_key_demo f
LEFT JOIN dim_customer_key_demo d ON f.customer_key = d.customer_key
WHERE d.customer_key IS NULL;
```

Expected: **0 rows** after valid inserts; failed inserts add no rows.

## Step 8 — Duplicate identity checks

```sql
SELECT source_system, source_customer_id, COUNT(*) AS row_count
FROM dim_customer_key_demo GROUP BY source_system, source_customer_id
HAVING COUNT(*) > 1;
```

Expected: 0 rows (UNIQUE already protects this).

```sql
SELECT order_id, line_number, COUNT(*) AS row_count
FROM fact_order_line_key_demo GROUP BY order_id, line_number
HAVING COUNT(*) > 1;
```

Expected: 0 rows.

## Step 9 — Unknown member design

When a fact arrives before its customer dimension:
- define a reserved warehouse key (documented Unknown member);
- assign a reason code such as `DIMENSION_NOT_ARRIVED`;
- count and alert on unknown-key facts;
- retain source customer identity in staging;
- reprocess and replace the unknown key after dimension arrival if policy requires;
- never use a random customer's key to satisfy a foreign key.

Document whether one generic Unknown row or multiple reason-specific members are used; this affects reporting.

## Deliverables

- [ ] Key strategy table: source identity, business key, surrogate key and fact key.
- [ ] SQL script and successful outputs.
- [ ] Evidence of duplicate source identity rejection.
- [ ] Evidence of duplicate order-line key rejection.
- [ ] Evidence of foreign-key violation.
- [ ] Orphan validation returning zero rows.
- [ ] Relationship diagram and unknown-member policy.
- [ ] README with assumptions and rerun steps.

## Answer key

- CRM 501 and Loyalty 501 are distinct source identities because the source system scopes the ID.
- customer_key is warehouse-generated and identifies a dimension row/version.
- (order_id, line_number) is composite because neither alone uniquely identifies a line.
- A fact foreign key prevents references to nonexistent dimension rows.
- An Unknown member preserves pipeline continuity only under documented, monitored, resolvable policy.

**Production note:** If records from different sources truly represent one person, use governed identity resolution/crosswalk logic. Never merge merely because names or numeric IDs match.
