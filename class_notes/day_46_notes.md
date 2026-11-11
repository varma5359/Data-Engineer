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

---

# Part 15 — Build one reconciliation report

Separate queries are useful for learning, but an engineer often needs one report that displays source and target differences together.

## 15.1 Why FULL OUTER JOIN helps

A FULL OUTER JOIN preserves:
- rows found only in the source;
- rows found only in the target;
- rows found in both.

Run this after the earlier training tables have been created and the target amount for order 1002 has been restored.

~~~sql
SELECT
    COALESCE(s.order_id, t.order_id) AS order_id,
    s.amount AS source_amount,
    t.amount AS target_amount,
    CASE
        WHEN s.order_id IS NULL THEN 'EXTRA_IN_TARGET'
        WHEN t.order_id IS NULL THEN 'MISSING_IN_TARGET'
        WHEN s.amount <> t.amount THEN 'AMOUNT_MISMATCH'
        ELSE 'MATCHED'
    END AS reconciliation_status
FROM (
    SELECT order_id, amount
    FROM source_orders
    WHERE status = 'PAID'
) AS s
FULL OUTER JOIN target_orders AS t
    ON s.order_id = t.order_id
ORDER BY COALESCE(s.order_id, t.order_id);
~~~

Expected result:

| order_id | source_amount | target_amount | reconciliation_status |
|---:|---:|---:|---|
| 1001 | 1000.00 | 1000.00 | MATCHED |
| 1002 | 1500.00 | 1500.00 | MATCHED |
| 1003 | 700.00 | 700.00 | MATCHED |

The pending order 1004 is excluded from the source subquery because it is not eligible for this target.

Understand the query:
- The source subquery selects only eligible PAID rows.
- The aliases s and t make the two inputs easier to reference.
- FULL OUTER JOIN keeps unmatched rows from either side.
- COALESCE returns the first non-NULL value, so the order ID is visible whichever side contains the row.
- CASE labels the comparison result.
- ORDER BY sorts the final report.

## 15.2 Create a missing-target scenario

Temporarily delete order 1002 from the target:

~~~sql
DELETE FROM target_orders
WHERE order_id = 1002;
~~~

Expected output:

~~~text
DELETE 1
~~~

Run the FULL OUTER JOIN query again. The row for order 1002 should show source_amount 1500.00, target_amount NULL and status MISSING_IN_TARGET.

Restore the row:

~~~sql
INSERT INTO target_orders (order_id, order_date, amount, status)
SELECT order_id, order_date, amount, status
FROM source_orders
WHERE order_id = 1002
  AND status = 'PAID';
~~~

Expected output:

~~~text
INSERT 0 1
~~~

## 15.3 Create an extra-target scenario

~~~sql
INSERT INTO target_orders VALUES
(1099, DATE '2026-01-03', 450.00, 'PAID');
~~~

Expected output: INSERT 0 1.

Run the reconciliation query. The extra row should show order_id 1099, source_amount NULL, target_amount 450.00 and status EXTRA_IN_TARGET.

Remove the intentionally incorrect row:

~~~sql
DELETE FROM target_orders
WHERE order_id = 1099;
~~~

Expected output: DELETE 1.

## 15.4 Create an amount mismatch

~~~sql
UPDATE target_orders
SET amount = 1550.00
WHERE order_id = 1002;
~~~

Expected output: UPDATE 1.

The reconciliation report should show order 1002 with source amount 1500.00, target amount 1550.00 and status AMOUNT_MISMATCH.

Restore the training value:

~~~sql
UPDATE target_orders
SET amount = 1500.00
WHERE order_id = 1002;
~~~

Expected output: UPDATE 1.

These are controlled learning changes. In production, investigate the root cause and use the approved reload or correction process. Do not casually edit production warehouse rows.

---

# Part 16 — Reconcile grouped totals with one query

A grand total can match even when one date or region is wrong. Grouped comparison helps locate the difference.

## 16.1 Compare daily totals

~~~sql
WITH source_daily AS (
    SELECT
        order_date,
        SUM(amount) AS source_total
    FROM source_orders
    WHERE status = 'PAID'
    GROUP BY order_date
),
target_daily AS (
    SELECT
        order_date,
        SUM(amount) AS target_total
    FROM target_orders
    GROUP BY order_date
)
SELECT
    COALESCE(s.order_date, t.order_date) AS order_date,
    s.source_total,
    t.target_total,
    COALESCE(s.source_total, 0) - COALESCE(t.target_total, 0) AS difference
FROM source_daily AS s
FULL OUTER JOIN target_daily AS t
    ON s.order_date = t.order_date
ORDER BY COALESCE(s.order_date, t.order_date);
~~~

Expected result after restoring the target:

| order_date | source_total | target_total | difference |
|---|---:|---:|---:|
| 2026-01-01 | 2500.00 | 2500.00 | 0.00 |
| 2026-01-02 | 700.00 | 700.00 | 0.00 |

A CTE is a named query result introduced with WITH:
- source_daily calculates daily source totals;
- target_daily calculates daily target totals;
- the final query joins the two summaries;
- difference is source total minus target total.

A CTE does not permanently create a table. It makes a multi-step query easier to read.

This example assumes order_date is the correct date role. If the business report uses payment date or shipment date, reconcile using that same date definition.

## 16.2 Understand a missing group

FULL OUTER JOIN preserves a date found on either side. COALESCE displays the date that exists.

For arithmetic, COALESCE(total, 0) treats an absent group as zero. This is appropriate for this comparison, but an absent group is not the same as a source value explicitly recorded as zero.

---

# Part 17 — Use a checksum carefully

A checksum is a compact value calculated from data. It can help detect whether row contents changed. It is supporting evidence, not a replacement for key-level reconciliation.

## 17.1 Create a row fingerprint

PostgreSQL provides MD5 for a text input. Run:

~~~sql
SELECT
    order_id,
    MD5(
        CONCAT_WS(
            '|',
            order_id::TEXT,
            order_date::TEXT,
            amount::TEXT,
            status
        )
    ) AS row_fingerprint
FROM source_orders
WHERE status = 'PAID'
ORDER BY order_id;
~~~

The exact fingerprint strings are generated by PostgreSQL. Inspect them in your SQL client rather than memorizing a fixed hash.

CONCAT_WS joins values using the supplied separator. Casting values to text gives MD5 a text input. MD5 returns a hexadecimal digest.

## 17.2 Checksum design warnings

A production fingerprint needs careful canonicalization:
- define column order;
- define date and timestamp formatting;
- define decimal formatting;
- define NULL representation;
- define text encoding and whitespace handling;
- avoid ambiguous separators inside values;
- include the fields whose changes matter.

A text value can itself contain the separator. Naive concatenation can then become ambiguous. A robust design uses unambiguous serialization or length-prefixing.

MD5 is not suitable for security or tamper-proof guarantees. For ordinary change detection, select a checksum approach appropriate to scale and risk. For security-sensitive integrity, use an appropriate cryptographic hash and trusted controls.

---

# Part 18 — Source snapshots, time windows and watermarks

A reconciliation is meaningful only when both sides refer to the same business scope.

## 18.1 What is a source snapshot?

A snapshot is a defined view of source data at a particular point or boundary.

Imagine:
- source extract begins at 07:00;
- a new order is inserted at 07:05;
- the warehouse query reads the source at 07:10;
- the target load was based on the 07:00 extract.

The source and target may differ even if the pipeline worked correctly. They represent different snapshots.

Record the extract boundary or batch ID and reconcile against that same source batch.

## 18.2 What is a watermark?

A watermark is a stored boundary used to track how far a pipeline has processed source data.

For a timestamp-based incremental load, it may be the maximum successfully processed source update time.

| Run | Previous watermark | New watermark |
|---|---|---|
| Run 1 | 2026-01-01 00:00 | 2026-01-02 00:00 |
| Run 2 | 2026-01-02 00:00 | 2026-01-03 00:00 |

A watermark is not automatically the latest business event time. It is a processing boundary defined by extraction logic.

## 18.3 Boundary conditions to document

When comparing source and target, document:
- inclusive or exclusive start boundary;
- inclusive or exclusive end boundary;
- time zone;
- timestamp precision;
- late-arriving update policy;
- deletion handling;
- source snapshot or batch identifier.

These filters are not equivalent:

~~~sql
WHERE updated_at > TIMESTAMP '2026-01-01 00:00:00'
~~~

~~~sql
WHERE updated_at >= TIMESTAMP '2026-01-01 00:00:00'
~~~

The second includes records exactly at midnight. If one side uses > and the other uses >=, boundary records may differ.

## 18.4 Why date-only comparisons can be dangerous

A timestamp such as 2026-01-02 00:30:00+00 may belong to a different local calendar date in another time zone.

For daily reconciliation:
1. choose the business time zone;
2. convert timestamps consistently;
3. define the start and end of the business day;
4. use the same interval for source and target;
5. record the time zone in the evidence.

---

# Part 19 — Lineage as an impact-analysis tool

Lineage is not only documentation. It helps engineers decide what to test when something changes.

## 19.1 Example: a source column is renamed

Suppose the source team renames order_total to gross_amount.

A lineage chain could be:

~~~text
orders.order_total
    -> raw_orders.order_total
    -> stg_orders.order_amount
    -> int_sales.sales_amount
    -> fct_sales.sales_amount
    -> sales_dashboard.total_sales
~~~

If the source field changes, inspect every dependent transformation and consumer.

## 19.2 Build an impact table

| Changed object | Direct dependent | Downstream consumer | Required action |
|---|---|---|---|
| orders.order_total | raw_orders | stg_orders | Update source mapping |
| stg_orders.order_amount | int_sales | fct_sales | Run model tests |
| fct_sales.sales_amount | sales_dashboard | Finance report | Reconcile business metric |

This is illustrative. A real dependency list must come from your repository, orchestration metadata or catalog—not from guessing.

## 19.3 Upstream and downstream

- Upstream: objects that provide data to the current object.
- Downstream: objects that depend on the current object.

For fct_sales, stg_orders may be upstream and sales_dashboard may be downstream.

When a source changes, trace downstream impact. When a report is wrong, trace upstream to find the source or transformation responsible.

## 19.4 Change-impact workflow

1. Identify the changed source table or column.
2. Find direct dependents.
3. Continue tracing downstream.
4. Identify affected reports and teams.
5. Review source-to-target mapping.
6. Update transformations and documentation.
7. Run relevant schema and data tests.
8. Reconcile affected outputs.
9. Communicate the change and evidence.

---

# Part 20 — Build a practical source-to-target mapping document

A mapping document should be specific enough that another engineer can understand and implement the transformation.

## 20.1 Example mapping for training orders

| Source field | Target field | Type conversion | Transformation | NULL policy | Validation |
|---|---|---|---|---|---|
| order_id | order_id | text to integer if needed | Trim and cast | Reject missing | Required and unique |
| order_dt | order_date | text to date | Parse using agreed format | Reject missing | Valid date |
| total | amount | text to numeric | Trim and decimal cast | Reject missing for PAID | Positive for PAID |
| state | status | text to varchar | Trim and uppercase | Reject missing | Approved values |

This is a teaching example. Real source field names and formats must be inspected.

## 20.2 Add ownership and change details

A useful mapping record also includes:
- source system and table;
- source owner;
- target model and column;
- target owner;
- business definition;
- transformation version;
- effective date;
- last reviewed date;
- sensitivity classification;
- related quality rule IDs.

## 20.3 Mapping versus lineage

Mapping says how a field is transformed from source to target.

Lineage shows the broader path and dependencies across systems and models.

A mapping is often one detailed edge in a larger lineage graph.

---

# Part 21 — Understand dbt lineage concepts

This course introduces dbt as a transformation and dependency tool. You do not need a production dbt project to understand the basic idea.

## 21.1 Declare a source

A dbt source can identify a raw database table.

~~~yaml
version: 2

sources:
  - name: raw
    schema: raw
    tables:
      - name: raw_orders
~~~

The database and schema must match your project configuration.

## 21.2 Create a staging model

A model file such as models/staging/stg_orders.sql can select from the declared source:

~~~sql
SELECT
    order_id,
    customer_id,
    order_date,
    amount,
    status
FROM {{ source('raw', 'raw_orders') }}
~~~

The source() expression tells dbt that this model depends on the declared source table.

## 21.3 Create a downstream model

~~~sql
SELECT
    order_id,
    customer_id,
    order_date,
    amount
FROM {{ ref('stg_orders') }}
WHERE status = 'PAID'
~~~

The ref() expression tells dbt that this model depends on stg_orders.

## 21.4 What does the dependency graph show?

~~~text
raw.raw_orders
      |
      v
stg_orders
      |
      v
fct_paid_orders
      |
      v
sales_dashboard
~~~

The dbt graph can show declared model dependencies. It cannot automatically discover every manual spreadsheet calculation, undocumented dashboard formula or business conversation. Keep those definitions documented too.

## 21.5 Where to inspect the graph

In a configured dbt project:
1. Open a terminal in the dbt project folder.
2. Run dbt docs generate after the project is configured and models are available.
3. Run dbt docs serve to start the local documentation site.
4. Open the local address printed by the command.
5. Select the graph view and inspect model dependencies.

These commands require a working dbt installation, project profile, database connection and valid configuration. If those are not ready, study the dependency examples first; do not interpret a setup error as a lineage error.

---

# Part 22 — Troubleshoot common reconciliation mismatches

| Symptom | Possible cause | First investigation |
|---|---|---|
| Source count higher than target | Missing records or filter mismatch | Compare source keys missing from target |
| Target count higher than source | Duplicate or extra records | Find target-only keys and duplicate keys |
| Counts match, totals differ | Amount or mapping error | Compare values by business key |
| Totals match, grouped totals differ | Errors offset across groups | Compare by date, region or category |
| Only one date differs | Boundary or late-arriving data | Inspect time zone and watermark |
| Many rows suddenly missing | Source schema/filter change | Check ingestion and transformation run |
| One dimension key is missing | Lookup or referential issue | Find unmatched dimension references |
| Reconciliation query errors | Schema or permission problem | Confirm object names, types and access |
| dbt graph omits a dependency | Dependency not declared or model bypasses dbt | Inspect source/ref declarations |

Do not assume the target is always wrong. A mismatch can originate from a source correction after extraction, target transformation defect, business filter difference, late-arriving event, timezone conversion, duplicate join, source deletion or rejection policy.

## 22.1 Keep a mismatch investigation record

| Field | Example |
|---|---|
| Incident ID | REC-2026-001 |
| Dataset | fct_sales |
| Run ID | run-2026-01-06-001 |
| Mismatch | Source has order 1002; target does not |
| Root cause | Load filter excluded an eligible status |
| Resolution | Correct filter and replay batch |
| Evidence | Key comparison returned zero missing keys after replay |
| Owner | Data Engineering |
| Closed at | Record actual resolution time |

These values illustrate the format, not a real incident.

---

# Part 23 — Practice exercises with worked answers

## Exercise 6 — Choose the right join

The source has keys 1, 2 and 3. The target has keys 1, 3 and 4. Which keys are missing and extra?

**Worked answer:**
- Key 2 is missing from the target.
- Key 4 is extra in the target.
- Keys 1 and 3 are present in both.
- A FULL OUTER JOIN or two anti-join queries can reveal these differences.

## Exercise 7 — Why use COALESCE?

A FULL OUTER JOIN returns a row that exists only in the target. The source key is NULL in that output row. How can one display the key?

**Worked answer:** Use COALESCE(source_key, target_key). It returns the first non-NULL key.

## Exercise 8 — Equal grand totals, different dates

Source totals are 500 for Monday and 700 for Tuesday. Target totals are 600 for Monday and 600 for Tuesday. Grand totals are both 1,200. Is reconciliation successful?

**Worked answer:** No. Daily totals differ. Grouped reconciliation reveals Monday is 100 too high and Tuesday is 100 too low.

## Exercise 9 — Source changed after extraction

The warehouse loaded a source snapshot at 07:00. At 07:10, the source corrected an order amount. At 07:15, you compare the live source with the target. Why might the comparison fail?

**Worked answer:** The source and target represent different snapshots. Reconcile against the extracted batch or record the source version/watermark used by the load.

## Exercise 10 — Hash values differ

Two row fingerprints differ. Does this automatically mean the business values changed?

**Worked answer:** Not necessarily. Date formatting, decimal formatting, NULL representation, whitespace or column ordering may differ. Canonicalize values before hashing, then compare actual fields for investigation.

## Exercise 11 — Find the owner

A dashboard total is incorrect because a source status mapping was changed without approval. Who should be involved?

**Worked answer:** The source/business owner should confirm the status meaning; the warehouse/model owner should correct the mapping; the report owner should validate the business metric. Record the change and test downstream outputs.

## Exercise 12 — Lineage direction

You are investigating why sales_dashboard.total_sales is wrong. Should you trace upstream or downstream?

**Worked answer:** Trace upstream from the dashboard metric to its model, transformations and source fields. If a source field changes, trace downstream to identify impacted reports.

## Exercise 13 — Missing dimension row

A fact row has product_key 900, but the product dimension has no matching key. What should you investigate?

**Worked answer:** Check whether the product is genuinely new, the dimension load failed, the fact arrived before its dimension, or the key mapping is wrong. Apply the warehouse's approved unknown-member or late-arriving-dimension policy.

---

# Part 24 — Day 46 practical completion checklist

### Reconciliation concepts
- [ ] Explain why matching row counts are not enough.
- [ ] Compare source and target using the same filters and time window.
- [ ] Explain row-count, key, aggregate, grouped and field-level reconciliation.
- [ ] Find missing source keys and extra target keys.
- [ ] Use FULL OUTER JOIN to display both sides.
- [ ] Explain why errors can cancel in grand totals.
- [ ] Compare grouped totals by date.
- [ ] Explain what a control total and run ID provide.
- [ ] Explain source snapshots and watermarks.
- [ ] Describe why timezone and boundary rules matter.

### SQL practice
- [ ] Create the source and target training tables.
- [ ] Load only eligible PAID rows.
- [ ] Compare counts and distinct keys.
- [ ] Find missing and extra records.
- [ ] Introduce and detect a changed amount.
- [ ] Build a FULL OUTER JOIN reconciliation report.
- [ ] Detect a missing-target record.
- [ ] Detect an extra-target record.
- [ ] Detect an amount mismatch.
- [ ] Compare daily source and target totals.
- [ ] Explain the purpose and limitations of a row fingerprint.

### Lineage and engineering
- [ ] Create a source-to-target mapping.
- [ ] Explain technical and business lineage.
- [ ] Distinguish upstream from downstream.
- [ ] Trace a report metric to source fields.
- [ ] Use source() and ref() to understand dbt dependencies.
- [ ] Perform a basic impact analysis after a column change.
- [ ] Record a mismatch, root cause, owner, resolution and evidence.

## Final self-assessment

Without looking at the notes, explain:

1. Why can two tables have equal row counts but different records?
2. Why can two tables have equal totals but incorrect individual values?
3. When is FULL OUTER JOIN useful for reconciliation?
4. Why must source and target use the same time boundary?
5. What does a watermark represent?
6. How is field mapping different from lineage?
7. How would you investigate a missing sales dashboard value?
8. What should you do before changing a source-to-target mapping?

If you can answer these questions and reproduce the SQL checks, you are ready to move to Day 47.

**Next lesson — Day 47: Metadata and Data Documentation.** We will build a data dictionary, business glossary and catalog entry, and learn how ownership, definitions, classifications and lineage make warehouse data discoverable and trustworthy.
