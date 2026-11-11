# Day 46: Data Reconciliation and Lineage

**Phase:** 5 — Data Warehousing and Modeling  
**Learning time:** 2–3 hours  
**Audience:** Complete beginners and junior Data Engineer candidates  
**Database:** PostgreSQL  
**Topic boundary:** Class notes only. Project workbook and interview guide are separate files.

> A warehouse table can contain the expected number of rows and still contain the wrong records. Reconciliation compares source and target evidence. Lineage explains where data came from and how it was transformed. Together they help a Data Engineer prove what was loaded, investigate differences and explain the impact of a change.

## Learning outcomes

By the end, you should be able to:
- explain reconciliation and lineage;
- distinguish count, key and aggregate reconciliation;
- create source-to-target mappings;
- compare source and target totals safely;
- identify missing, extra and changed records;
- understand control totals and audit metadata;
- explain technical and business lineage;
- trace a report value to upstream models;
- document transformations and dependencies;
- investigate common mismatch scenarios;
- understand reconciliation limitations.

## Today's flow

`Source snapshot -> Mapping -> Transformation -> Target -> Counts/keys/totals -> Difference analysis -> Lineage trace -> Resolution`

---

# Part 1 — Why reconciliation matters

## 1.1 Business scenario

TechStore extracts 1,000 paid order lines from a source database.

The warehouse reports that 1,000 rows were loaded.

Is the load correct?

Not necessarily.

Possible defects:
- one source row is missing and one duplicate was loaded;
- one amount was changed;
- a filter excluded a valid row and included an invalid row;
- all rows loaded but one column was mapped incorrectly;
- source and target use different time boundaries.

A matching row count is only one piece of evidence.

## 1.2 What is reconciliation?

Reconciliation is the process of comparing related datasets or measures to confirm that expected records and values have been transferred or transformed correctly.

It may compare:
- source and target row counts;
- distinct business keys;
- missing and extra keys;
- sums and balances;
- grouped totals;
- field-level values;
- control totals;
- time windows and watermarks.

## 1.3 What is lineage?

Data lineage describes where data originated, how it moved and which transformations or models produced it.

Example:

`orders database -> raw_orders -> stg_orders -> int_sales -> fct_sales -> sales dashboard`

Lineage helps answer:
- Where did this number come from?
- Which source table feeds this report?
- Which models depend on a changed column?
- Which reports may be affected by a failed pipeline?

---

# Part 2 — Source-to-target mapping

## 2.1 What is source-to-target mapping?

It documents how each source field becomes a target field.

Example:

| Source | Source field | Target | Target field | Transformation |
|---|---|---|---|---|
| order_db.orders | order_no | stg_orders | order_id | Rename and cast |
| order_db.orders | order_dt | stg_orders | order_date | Parse as date |
| order_db.orders | total | stg_orders | order_amount | Decimal conversion |
| order_db.orders | state | stg_orders | order_status | Trim and uppercase |

## 2.2 Why mapping is important

Without mapping:
- engineers may use the wrong source column;
- business meaning can be lost;
- type conversions may be hidden;
- debugging requires guesswork;
- schema changes are harder to assess.

A mapping should also record:
- source owner;
- target owner;
- data type;
- null behavior;
- key role;
- sensitivity;
- transformation rule;
- quality rule.

## 2.3 Example mapping decision

Source field `total` contains text values such as "2500.00".

Target field `order_amount` is NUMERIC(12,2).

The transformation must:
1. trim whitespace;
2. validate the numeric representation;
3. convert to decimal;
4. reject or quarantine invalid values;
5. preserve source value for investigation.

Do not silently convert invalid text to zero.

---

# Part 3 — Reconciliation methods

## 3.1 Row-count reconciliation

Compare the number of records in the source scope and target scope.

Example:
- source eligible rows = 1,000;
- target loaded rows = 1,000.

This is useful but not sufficient.

## 3.2 Distinct-key reconciliation

Compare unique business identifiers.

This helps detect:
- missing keys;
- extra keys;
- duplicate keys.

## 3.3 Aggregate reconciliation

Compare totals such as:
- order amount;
- quantity;
- tax;
- payment amount;
- refund amount.

The source and target must use the same filters, currency, date range and business definition.

## 3.4 Grouped reconciliation

Compare totals by:
- day;
- region;
- category;
- source system;
- status.

A total can match overall while one region is overstated and another understated.

Grouped reconciliation can reveal the mismatch.

## 3.5 Field-level reconciliation

Compare individual mapped values for the same business key.

This is useful when counts and totals match but values may be mapped incorrectly.

## 3.6 Reconciliation matrix

| Check | Detects | Limitation |
|---|---|---|
| Row count | Missing/extra row count | Cannot identify swapped rows |
| Distinct keys | Missing/extra identities | Does not validate all values |
| Sum totals | Aggregate differences | Errors can offset |
| Grouped totals | Segment-level differences | Still not row-level proof |
| Field comparison | Value differences | Can be expensive at large scale |
| Hash/checksum | Potential row-content changes | Requires canonicalization and careful design |

Use several checks together.

---

# Part 4 — Create source and target training tables

Run in one PostgreSQL session.

## 4.1 Create source table

```sql
CREATE TEMP TABLE source_orders (
    order_id INTEGER PRIMARY KEY,
    order_date DATE NOT NULL,
    amount NUMERIC(10, 2) NOT NULL,
    status VARCHAR(20) NOT NULL
);
```

Expected output:

```text
CREATE TABLE
```

## 4.2 Insert source rows

```sql
INSERT INTO source_orders VALUES
(1001, DATE '2026-01-01', 1000.00, 'PAID'),
(1002, DATE '2026-01-01', 1500.00, 'PAID'),
(1003, DATE '2026-01-02', 700.00, 'PAID'),
(1004, DATE '2026-01-02', 300.00, 'PENDING');
```

Expected output:

```text
INSERT 0 4
```

## 4.3 Create target table

```sql
CREATE TEMP TABLE target_orders (
    order_id INTEGER PRIMARY KEY,
    order_date DATE NOT NULL,
    amount NUMERIC(10, 2) NOT NULL,
    status VARCHAR(20) NOT NULL
);
```

Expected output:

```text
CREATE TABLE
```

## 4.4 Insert the eligible source rows into target

For this exercise, only PAID rows should be loaded.

```sql
INSERT INTO target_orders
SELECT order_id, order_date, amount, status
FROM source_orders
WHERE status = 'PAID';
```

Expected output:

```text
INSERT 0 3
```

---

# Part 5 — Run reconciliation checks

## 5.1 Compare expected eligible count

```sql
SELECT COUNT(*) AS source_eligible_count
FROM source_orders
WHERE status = 'PAID';
```

Expected result:

| source_eligible_count |
|---:|
| 3 |

Target count:

```sql
SELECT COUNT(*) AS target_count
FROM target_orders;
```

Expected result:

| target_count |
|---:|
| 3 |

The counts match.

## 5.2 Compare distinct keys

```sql
SELECT COUNT(DISTINCT order_id) AS source_distinct_keys
FROM source_orders
WHERE status = 'PAID';
```

Expected result: 3.

```sql
SELECT COUNT(DISTINCT order_id) AS target_distinct_keys
FROM target_orders;
```

Expected result: 3.

## 5.3 Find source keys missing from target

```sql
SELECT s.order_id
FROM source_orders AS s
LEFT JOIN target_orders AS t
    ON s.order_id = t.order_id
WHERE s.status = 'PAID'
  AND t.order_id IS NULL;
```

Expected result: zero rows.

## 5.4 Find extra target keys

```sql
SELECT t.order_id
FROM target_orders AS t
LEFT JOIN source_orders AS s
    ON t.order_id = s.order_id
WHERE s.order_id IS NULL
   OR s.status <> 'PAID';
```

Expected result: zero rows.

This checks that target contains no missing-source or ineligible records.

## 5.5 Compare amount totals

Source eligible total:

```sql
SELECT SUM(amount) AS source_paid_amount
FROM source_orders
WHERE status = 'PAID';
```

Expected result:

| source_paid_amount |
|---:|
| 3200.00 |

Target total:

```sql
SELECT SUM(amount) AS target_paid_amount
FROM target_orders;
```

Expected result:

| target_paid_amount |
|---:|
| 3200.00 |

The totals match.

## 5.6 Compare totals by date

```sql
SELECT
    order_date,
    SUM(amount) AS source_amount
FROM source_orders
WHERE status = 'PAID'
GROUP BY order_date
ORDER BY order_date;
```

Expected result:

| order_date | source_amount |
|---|---:|
| 2026-01-01 | 2500.00 |
| 2026-01-02 | 700.00 |

Run the equivalent query against target.

Expected result should match exactly.

Grouped reconciliation is more informative than only comparing the grand total.

---

# Part 6 — Find changed values

Counts and totals can match even when individual values differ.

## 6.1 Introduce a controlled mismatch

```sql
UPDATE target_orders
SET amount = 1600.00
WHERE order_id = 1002;
```

Expected output:

```text
UPDATE 1
```

The target amount now differs from source.

## 6.2 Compare fields by business key

```sql
SELECT
    s.order_id,
    s.amount AS source_amount,
    t.amount AS target_amount
FROM source_orders AS s
JOIN target_orders AS t
    ON s.order_id = t.order_id
WHERE s.status = 'PAID'
  AND s.amount <> t.amount;
```

Expected result:

| order_id | source_amount | target_amount |
|---:|---:|---:|
| 1002 | 1500.00 | 1600.00 |

This identifies the exact mismatched record.

Restore the training value:

```sql
UPDATE target_orders
SET amount = 1500.00
WHERE order_id = 1002;
```

Expected output:

```text
UPDATE 1
```

---

# Part 7 — Control totals and audit metadata

## 7.1 What is a control total?

A control total is a value recorded at a processing boundary to support comparison.

Examples:
- number of source rows;
- sum of source amount;
- number of accepted rows;
- number of rejected rows;
- number of target rows.

A control total is useful only when its scope and calculation are documented.

## 7.2 Example run control record

| run_id | source_count | accepted_count | rejected_count | target_count | status |
|---|---:|---:|---:|---:|---|
| run-001 | 1000 | 970 | 30 | 970 | PASSED |

The counts reconcile:
1,000 = 970 + 30.

But the engineer should also validate target keys and values.

## 7.3 Audit metadata

A warehouse load may store:
- run ID;
- source system;
- source file or batch;
- extracted timestamp;
- loaded timestamp;
- source watermark;
- row counts;
- model version;
- status.

This makes the load explainable.

## 7.4 Why audit metadata matters

When a business user asks why a report changed, the engineer can identify:
- which source batch was loaded;
- which transformation version ran;
- when the target changed;
- how many rows were accepted;
- which quality checks passed.

---

# Part 8 — What is data lineage?

## 8.1 Technical lineage

Technical lineage traces tables, columns, files, models and jobs.

Example:

`source_orders.amount -> raw_orders.amount -> stg_orders.order_amount -> fct_sales.line_sales_amount -> sales_dashboard`

## 8.2 Business lineage

Business lineage explains how a business concept is derived.

Example:

**Paid Sales**
- includes eligible paid order lines;
- excludes pending and cancelled orders;
- uses order date;
- sums approved line amount;
- does not subtract refunds unless a separate net-sales definition is selected.

Business lineage helps managers understand what a metric means.

## 8.3 Table-level lineage versus column-level lineage

Table-level:
- fct_sales depends on stg_orders and dim_product.

Column-level:
- fct_sales.line_sales_amount is derived from stg_order_items.quantity and unit_price.

Column-level lineage is more precise but requires richer metadata and tooling.

## 8.4 Why lineage matters

- impact analysis;
- root-cause analysis;
- compliance and audit;
- change management;
- trust in reports;
- ownership and accountability.

---

# Part 9 — Build a simple lineage register

Create a documentation table in your project repository.

| Target model | Target column | Upstream source | Transformation | Business meaning |
|---|---|---|---|---|
| fct_sales | order_id | raw_orders.order_no | Rename and cast | Source order identity |
| fct_sales | date_key | raw_orders.order_dt | Date lookup | Order date dimension key |
| fct_sales | line_sales_amount | raw_order_items | quantity * unit_price | Sales value per line |
| dim_product | category | raw_products.category_name | Standardize mapping | Approved product category |

A lineage register should also identify:
- model owner;
- last reviewed date;
- downstream consumers;
- sensitive data classification.

---

# Part 10 — Lineage and dbt

dbt models use dependencies such as `ref()` and declared sources.

Example:

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    amount
FROM {{ source('raw', 'raw_orders') }}
```

A downstream model:

```sql
SELECT *
FROM {{ ref('stg_orders') }}
```

dbt can use these dependencies to build a model graph.

The graph is useful, but it does not automatically explain every business rule. Add model and column descriptions, tests and business definitions.

---

# Part 11 — Reconciliation failure investigation

Use this order:

1. Confirm source and target scope.
2. Confirm filters and date boundaries.
3. Compare source and target counts.
4. Compare distinct business keys.
5. Find missing source keys.
6. Find extra target keys.
7. Compare aggregate totals.
8. Compare grouped totals.
9. Compare field values for mismatched keys.
10. Inspect transformation logic.
11. Inspect late arrivals and duplicate events.
12. Record root cause and correction.
13. Re-run reconciliation.
14. Preserve evidence.

Do not update target values manually without documenting the source of truth and correction.

---

# Part 12 — Common mistakes

| Mistake | Why it fails | Better approach |
|---|---|---|
| Compare counts only | Different rows can produce the same count | Compare keys and values |
| Compare totals with different filters | The result is meaningless | Align scope and definitions |
| Ignore date/time zones | Boundary rows may be excluded | Document time zone and interval |
| Treat equal totals as proof | Errors can offset | Add row/key/group checks |
| No source-to-target mapping | Transformations are hard to trace | Maintain field-level mapping |
| Only table lineage | Column logic remains unclear | Document important column lineage |
| No run metadata | Cannot identify which batch produced data | Store run and batch identifiers |
| Fix mismatches manually | Root cause remains and audit is lost | Record approved correction and retest |

---

# Part 13 — Practice exercises

## Exercise 1

Source has 500 eligible records. Target has 500 records. Is reconciliation complete?

**Answer:** No. Compare distinct keys, missing/extra rows, important values and grouped totals.

## Exercise 2

Source amount total is 100,000 and target amount total is 100,000, but one row is 10,000 too high and another is 10,000 too low. Is the target correct?

**Answer:** No. Aggregate errors cancel. Use key-level comparison and grouped checks.

## Exercise 3

A report column comes from a source field after two transformations. What should lineage record?

**Answer:** Source field, intermediate models, transformation logic, final target field and business definition.

## Exercise 4

Why record a run ID with reconciliation results?

**Answer:** It connects the comparison to the exact source batch, target load and pipeline execution.

## Exercise 5

What is the difference between technical and business lineage?

**Answer:** Technical lineage traces systems, tables, columns and jobs. Business lineage explains the meaning and rules behind a business metric or data element.

---

# Part 14 — Revision map

| Term | Remember |
|---|---|
| Reconciliation | Compare source and target evidence |
| Source-to-target mapping | Field-by-field transformation specification |
| Control total | Recorded count or amount used for comparison |
| Orphan key | Key with no matching parent/dimension |
| Missing key | Source identity absent from target |
| Extra key | Target identity absent from eligible source |
| Technical lineage | Systems, jobs, tables and columns |
| Business lineage | Business meaning and derivation |
| Run metadata | Evidence about one pipeline execution |
| Impact analysis | Identify downstream objects affected by a change |

## Final competency checklist

- [ ] Explain reconciliation and lineage.
- [ ] Create source-to-target mappings.
- [ ] Compare counts, keys and amounts.
- [ ] Find missing and extra keys.
- [ ] Detect changed values.
- [ ] Explain why equal totals are not enough.
- [ ] Record run-level control totals.
- [ ] Explain technical and business lineage.
- [ ] Trace a report metric to its source.
- [ ] Document a mismatch investigation.

**Next lesson — Day 47: Metadata and Data Documentation.** We will build a data dictionary, business glossary and catalog entry, and learn how ownership, definitions, classifications and lineage make warehouse data discoverable and trustworthy.
