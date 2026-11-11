# Day 45: Data Quality and Validation

**Phase:** 5 — Data Warehousing and Modeling  
**Learning time:** 2–3 hours  
**Audience:** Complete beginners and junior Data Engineer candidates  
**Database:** PostgreSQL  
**Topic boundary:** Class notes only. Project workbook and interview guide are separate files.

> A pipeline can run successfully, load every row and still publish incorrect data. Data quality is the discipline of checking whether data is fit for its intended use. Today we learn the main quality dimensions, turn business expectations into rules, write SQL validation checks, understand rejected records and design a quality gate before data reaches reports.

## Learning outcomes

By the end, you should be able to:
- explain data quality in business terms;
- distinguish accuracy, completeness, consistency, validity, uniqueness and freshness;
- translate a business expectation into a testable rule;
- write SQL checks for nulls, duplicates, ranges and allowed values;
- understand referential integrity checks;
- distinguish warning and blocking failures;
- preserve rejected records and reasons;
- build a basic quality summary;
- understand thresholds and false positives;
- design a quality gate for a warehouse model;
- explain why technical success is not data correctness.

## Today's flow

`Business expectation -> Quality dimension -> Rule -> Test -> Result -> Threshold -> Alert/block -> Correction -> Re-test`

---

# Part 1 — What is data quality?

## 1.1 Beginner definition

Data quality describes how well data meets the requirements of its intended use.

A customer address may be acceptable for a rough regional report but not acceptable for shipping a package.

Quality depends on the business purpose.

A Data Engineer should not simply say "the data looks clean". They should identify the expectation and show evidence that it passed.

## 1.2 Example: TechStore order data

A sales dashboard receives these rows:

| order_id | customer_id | order_date | amount | status |
|---:|---:|---|---:|---|
| 1001 | 501 | 2026-01-02 | 2500 | PAID |
| 1002 | NULL | 2026-01-03 | 900 | PAID |
| 1002 | 502 | 2026-01-03 | 900 | PAID |
| 1004 | 503 | 2026-01-04 | -500 | PAID |
| 1005 | 504 | 2026-01-05 | 1200 | UNKNOWN |

Possible issues:
- missing customer ID;
- duplicate order ID;
- negative amount;
- unapproved status.

The pipeline may still load all five rows. That does not mean the data is ready for reporting.

## 1.3 Quality is not the same as perfection

Some fields can be legitimately optional.

For example:
- a guest order may not have a registered customer ID;
- a refund may have a negative amount under a particular convention;
- an order may have a status such as PENDING.

Do not write a test just because a value looks unusual.

First understand the business rule.

---

# Part 2 — Six important quality dimensions

## 2.1 Accuracy

Accuracy asks: does the value correctly represent the real-world fact?

Example:
- source says customer city is Hyderabad;
- actual verified city is Bengaluru.

The value is inaccurate if the business fact is Bengaluru.

A database cannot prove accuracy by checking only data type or null status. Accuracy often requires a trusted source, reconciliation or external verification.

## 2.2 Completeness

Completeness asks: are required values or records present?

Examples:
- order ID must be present;
- every expected daily source file must arrive;
- every paid order should have a payment reference under the business rule.

Completeness can be measured at field, record, table and source-delivery levels.

## 2.3 Consistency

Consistency asks: do related values agree across records and systems?

Examples:
- order says PAID but payment system says FAILED;
- customer ID maps to two different active identities;
- one report says a category is Electronics while another says Home.

Consistency requires comparison across related data and shared definitions.

## 2.4 Validity

Validity asks: does the value follow an allowed format, type, range or rule?

Examples:
- email follows the agreed basic format;
- quantity is a positive integer;
- status belongs to an approved list;
- order date is not beyond an allowed future boundary.

A value can be valid in format but inaccurate in reality.

For example, a syntactically valid email may not belong to the customer.

## 2.5 Uniqueness

Uniqueness asks: does a record appear only once according to its business identity?

Examples:
- order_id should be unique in an order-header table;
- (order_id, line_number) should be unique in an order-line table;
- a source event ID should not be loaded twice.

Do not confuse duplicate rows with duplicate values in a field that is allowed to repeat.

Many orders can have the same customer_id. That is not a duplicate-order defect.

## 2.6 Freshness

Freshness asks: how recently does the data reflect the source?

Examples:
- daily sales data should be updated by 8 AM;
- inventory data should be no more than 15 minutes old;
- a payment feed should arrive within its delivery window.

A table can be accurate but stale.

## 2.7 Summary

| Dimension | Main question | Example check |
|---|---|---|
| Accuracy | Is it correct? | Reconcile amount to trusted source |
| Completeness | Is required data present? | Required field not null |
| Consistency | Do related values agree? | Payment/order status comparison |
| Validity | Does it follow rules? | Status in allowed list |
| Uniqueness | Is identity duplicated? | Unique business key |
| Freshness | Is it recent enough? | Latest source timestamp |

---

# Part 3 — Convert expectations into rules

## 3.1 Start with a requirement

Business statement:

"Every paid order must have a positive amount and a customer reference."

Convert it into rules:

1. status = PAID;
2. amount must be greater than zero;
3. customer reference must be present.

The test should be clear enough that two engineers would implement it the same way.

## 3.2 Define rule metadata

For every important check, document:
- rule ID;
- dataset/table;
- column or relationship;
- business expectation;
- test logic;
- severity;
- owner;
- failure action;
- exception policy.

Example:

| Field | Value |
|---|---|
| Rule ID | ORD-001 |
| Dataset | stg_orders |
| Column | order_id |
| Expectation | Required and unique |
| Severity | Blocking |
| Failure action | Do not publish sales mart |
| Owner | Order data owner |

## 3.3 Blocking versus warning checks

A blocking check prevents publication when the failure makes the output unsafe.

Examples:
- duplicate primary business key;
- missing required source;
- failed reconciliation;
- impossible amount under the approved rule.

A warning check reports a problem but allows controlled continuation.

Examples:
- optional description missing;
- small increase in non-critical rejection rate;
- a new but approved informational category.

Severity must be agreed with the business owner.

Do not make every warning a failure. Do not make critical failures warnings just to keep a green dashboard.

---

# Part 4 — Create a training dataset

Run in PostgreSQL.

```sql
CREATE TEMP TABLE quality_orders (
    order_id INTEGER,
    customer_id INTEGER,
    order_date DATE,
    amount NUMERIC(10, 2),
    status VARCHAR(20)
);
```

Expected output:

```text
CREATE TABLE
```

Insert training rows:

```sql
INSERT INTO quality_orders VALUES
(1001, 501, DATE '2026-01-02', 2500.00, 'PAID'),
(1002, NULL, DATE '2026-01-03', 900.00, 'PAID'),
(1002, 502, DATE '2026-01-03', 900.00, 'PAID'),
(1004, 503, DATE '2026-01-04', -500.00, 'PAID'),
(1005, 504, DATE '2026-01-05', 1200.00, 'UNKNOWN'),
(1006, 505, DATE '2026-01-06', 0.00, 'PENDING');
```

Expected output:

```text
INSERT 0 6
```

Inspect:

```sql
SELECT *
FROM quality_orders
ORDER BY order_id;
```

There are six rows. We intentionally inserted quality problems for learning.

---

# Part 5 — Completeness checks

## 5.1 Find missing required order IDs

```sql
SELECT *
FROM quality_orders
WHERE order_id IS NULL;
```

Expected result: zero rows.

## 5.2 Find missing customer IDs

```sql
SELECT *
FROM quality_orders
WHERE customer_id IS NULL;
```

Expected result:

| order_id | customer_id | order_date | amount | status |
|---:|---|---|---:|---|
| 1002 | NULL | 2026-01-03 | 900.00 | PAID |

One row has a missing customer reference.

## 5.3 Count missing values

```sql
SELECT
    COUNT(*) AS total_rows,
    COUNT(customer_id) AS rows_with_customer,
    COUNT(*) - COUNT(customer_id) AS missing_customer_rows
FROM quality_orders;
```

Expected result:

| total_rows | rows_with_customer | missing_customer_rows |
|---:|---:|---:|
| 6 | 5 | 1 |

COUNT(column) ignores NULL values. COUNT(*) counts rows.

## 5.4 Is a missing customer always invalid?

Not necessarily. A guest checkout may have no registered customer ID.

The business rule must decide whether:
- guest orders are allowed;
- a guest customer dimension member is used;
- customer_id is required only for registered orders.

A test should reflect the actual contract.

---

# Part 6 — Uniqueness checks

## 6.1 Find duplicate order IDs

```sql
SELECT
    order_id,
    COUNT(*) AS row_count
FROM quality_orders
GROUP BY order_id
HAVING COUNT(*) > 1;
```

Expected result:

| order_id | row_count |
|---:|---:|
| 1002 | 2 |

The source table may intentionally contain multiple order lines per order. In that case order_id is not unique at line grain.

For this training table, the intended grain is one row per order header, so the duplicate is a defect.

## 6.2 Check a composite key

If the table is at order-line grain, test the pair order_id and line_number.

```sql
SELECT
    order_id,
    line_number,
    COUNT(*) AS row_count
FROM order_items
GROUP BY order_id, line_number
HAVING COUNT(*) > 1;
```

This example assumes an order_items table with those columns. Use the actual source schema.

## 6.3 Do not check the wrong key

customer_id repeats because one customer can place many orders.

This is not a uniqueness defect in an order table.

The correct key depends on table grain.

---

# Part 7 — Validity checks

## 7.1 Check allowed status values

```sql
SELECT *
FROM quality_orders
WHERE status NOT IN ('PAID', 'PENDING', 'CANCELLED');
```

Expected result:

| order_id | customer_id | order_date | amount | status |
|---:|---:|---|---:|---|
| 1005 | 504 | 2026-01-05 | 1200.00 | UNKNOWN |

This identifies the unapproved status.

If status can be NULL, handle NULL separately because SQL comparisons with NULL evaluate to unknown.

## 7.2 Check positive paid amount

```sql
SELECT *
FROM quality_orders
WHERE status = 'PAID'
  AND amount <= 0;
```

Expected result:

| order_id | customer_id | order_date | amount | status |
|---:|---:|---|---:|---|
| 1004 | 503 | 2026-01-04 | -500.00 | PAID |

The negative amount violates the training rule.

A refund table may legitimately contain negative values. Do not apply an order rule to a refund fact.

## 7.3 Check date boundaries

```sql
SELECT *
FROM quality_orders
WHERE order_date > CURRENT_DATE;
```

Expected output depends on the date when the query is run. This is a dynamic check, so do not write a fixed expected row count.

A production rule may allow future-dated orders for preorders. Clarify the business rule.

## 7.4 Check text length or format

Example: a status field should not be blank.

```sql
SELECT *
FROM quality_orders
WHERE status IS NULL
   OR TRIM(status) = '';
```

Expected result: zero rows in this fixture.

---

# Part 8 — Consistency checks

## 8.1 Compare related business states

Suppose an order says PAID but no successful payment exists.

A consistency check may compare the order and payment models.

Conceptual query:

```sql
SELECT o.order_id
FROM stg_orders AS o
LEFT JOIN stg_payments AS p
    ON o.order_id = p.order_id
   AND p.payment_status = 'SUCCESS'
WHERE o.order_status = 'PAID'
  AND p.order_id IS NULL;
```

This is a conceptual example. It assumes those staging models and columns exist.

The result identifies paid orders without a matching successful payment.

Before treating this as an error, clarify:
- split payments;
- delayed settlement;
- payment reversals;
- cash-on-delivery;
- source update timing.

## 8.2 Cross-report consistency

If two reports calculate sales differently, compare:
- filters;
- date roles;
- grain;
- refund treatment;
- currency;
- source freshness;
- metric definitions.

A consistency test should compare equivalent scopes, not simply compare two totals with different definitions.

---

# Part 9 — Accuracy and reconciliation

Accuracy is difficult to prove with only the target table.

Use a trusted reference where possible.

Examples:
- source transaction total;
- finance-approved control total;
- payment settlement statement;
- independently calculated sample.

For a source-to-target comparison, align:
- same time window;
- same status filters;
- same currency;
- same grain;
- same late-arrival cutoff.

A matching total is useful evidence but does not prove every row is correct. Two wrong values can cancel each other.

Use multiple checks:
- counts;
- unique keys;
- amount totals;
- sampled records;
- business invariants.

---

# Part 10 — Freshness checks

## 10.1 What is freshness?

Freshness measures how recently the target reflects the source according to a defined expectation.

Example:
- source last updated at 07:30;
- warehouse load completed at 07:50;
- dashboard published at 08:00.

The data may meet an 08:00 deadline.

## 10.2 Query latest data timestamp

```sql
SELECT MAX(order_date) AS latest_order_date
FROM quality_orders;
```

Expected result for the fixed fixture:

| latest_order_date |
|---|
| 2026-01-06 |

This checks the latest business date in the fixture, not the pipeline's last successful run time.

## 10.3 Business freshness versus technical freshness

- Business freshness: how recent are the events represented?
- Technical freshness: when did the pipeline last load successfully?

Both matter.

A pipeline can run now but load old source data.

## 10.4 Freshness SLA example

Requirement:
"Sales mart must contain all source orders received by 07:00 and publish by 08:00."

A good monitor should check:
- source delivery completed;
- extraction reached the expected watermark;
- warehouse load completed;
- quality checks passed;
- mart publication completed.

A job status alone is not sufficient.

---

# Part 11 — Build a quality summary

## 11.1 Count records by status

```sql
SELECT
    status,
    COUNT(*) AS row_count
FROM quality_orders
GROUP BY status
ORDER BY status;
```

Expected result:

| status | row_count |
|---|---:|
| PAID | 4 |
| PENDING | 1 |
| UNKNOWN | 1 |

## 11.2 Count quality exceptions

```sql
SELECT
    SUM(CASE WHEN customer_id IS NULL THEN 1 ELSE 0 END) AS missing_customer,
    SUM(CASE WHEN amount <= 0 AND status = 'PAID' THEN 1 ELSE 0 END) AS invalid_paid_amount,
    SUM(CASE WHEN status NOT IN ('PAID', 'PENDING', 'CANCELLED') THEN 1 ELSE 0 END) AS invalid_status
FROM quality_orders;
```

Expected result:

| missing_customer | invalid_paid_amount | invalid_status |
|---:|---:|---:|
| 1 | 1 | 1 |

This is a compact summary. It does not replace detailed rejected-row evidence.

## 11.3 Store quality results

A production system may write test results to an audit table with:
- run_id;
- dataset;
- rule_id;
- evaluated_at;
- failed_count;
- total_count;
- severity;
- status.

This makes quality trends measurable over time.

---

# Part 12 — Reject, quarantine or block?

A failed quality rule needs an action.

Possible actions:

1. **Reject the record:** Do not allow it into the accepted dataset.
2. **Quarantine the record:** Preserve it separately with a reason for investigation.
3. **Block the batch:** Prevent publication if the dataset is unsafe.
4. **Warn:** Allow publication while reporting a non-critical issue.
5. **Correct under an approved rule:** Apply a documented transformation and retain audit evidence.

Do not delete invalid records silently.

## 12.1 Example rejection reason

| order_id | rejected_field | rejected_value | reason |
|---:|---|---|---|
| 1004 | amount | -500.00 | PAID_ORDER_AMOUNT_MUST_BE_POSITIVE |
| 1005 | status | UNKNOWN | STATUS_NOT_IN_ALLOWED_SET |

A rejection table should retain enough safe context to investigate without exposing secrets or unnecessary personal information.

---

# Part 13 — Data quality in dbt

dbt supports tests on models and columns. Exact syntax depends on dbt version and project configuration.

Common built-in test concepts:
- unique;
- not_null;
- accepted_values;
- relationships.

Example YAML:

```yaml
version: 2

models:
  - name: stg_orders
    columns:
      - name: order_id
        data_tests:
          - not_null
          - unique

      - name: status
        data_tests:
          - accepted_values:
              arguments:
                values: ['PAID', 'PENDING', 'CANCELLED']
```

Some earlier dbt project versions use `tests:` instead of `data_tests:`. Use the syntax supported by the installed dbt version and verify it with the project's documentation and `dbt parse`.

A relationship test can ensure every non-null customer key matches a row in the referenced dimension.

A custom SQL test can encode a business rule such as positive paid order amounts.

## 13.1 What does a passing dbt test mean?

It means the tested condition passed for the evaluated data at that time.

It does not prove:
- every business definition is correct;
- source data is accurate;
- every row is complete;
- the pipeline is fresh;
- all downstream reports are correct.

Tests cover explicit expectations only.

---

# Part 14 — Quality gates

A quality gate is a decision point that determines whether data can move to the next stage.

Example:

`Raw -> Structural checks -> Staging -> Business checks -> Curated mart -> BI`

Possible policy:
- missing required source: block;
- duplicate order key: block;
- unknown optional product description: warn;
- critical reconciliation mismatch: block;
- freshness beyond agreed limit: mark stale and alert.

The policy must be documented and agreed with data owners.

## 14.1 Avoid alert fatigue

An alert should be:
- actionable;
- assigned to an owner;
- linked to evidence;
- deduplicated where possible;
- prioritized by business impact.

Do not send an urgent alert for every harmless missing optional description.

---

# Part 15 — Common mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Checking only NULLs | Misses wrong values and duplicates | Use multiple quality dimensions |
| Treating all missing values as errors | Some fields are optional | Confirm business contract |
| Using DISTINCT to hide duplicates | Hides source or join defect | Test the correct business key |
| Replacing invalid amounts with zero | Changes business meaning | Quarantine or apply approved rule |
| Treating latest job time as data freshness | Source may be stale | Compare source event/update timestamps |
| Treating one matching total as full accuracy proof | Errors can offset | Combine counts, keys, totals and samples |
| Making all tests blocking | Creates unnecessary outages | Define severity and thresholds |
| Making all tests warnings | Bad data reaches users | Block critical defects |
| Testing wrong grain | False failures or missed duplicates | Define grain before key tests |
| Logging rejected personal data | Privacy/security risk | Store only necessary protected context |

---

# Part 16 — Practice exercises

## Exercise 1

A customer table has a missing optional middle_name.

Which dimension may be affected?

**Answer:** Completeness, but whether it is a failure depends on whether middle_name is required.

## Exercise 2

Two order-header rows have the same order_id.

Which dimension?

**Answer:** Uniqueness, assuming order_id is the intended key and the table grain is one row per order.

## Exercise 3

A paid order has amount -100.

Which rule?

**Answer:** Validity/business-rule check. Under today's training rule, paid order amount must be positive.

## Exercise 4

A source feed has not updated for 36 hours, but the pipeline ran successfully five minutes ago.

Which quality dimension is at risk?

**Answer:** Freshness. Technical run time does not prove source data is recent.

## Exercise 5

A source-to-target total matches, but the target has duplicate rows and missing rows that cancel each other in the total.

Is the data correct?

**Answer:** No. Aggregate agreement alone does not prove row-level correctness. Validate keys, counts, samples and business rules.

## Exercise 6

A new order status appears: ON_HOLD.

What should the engineer do?

**Answer:** Check the source contract and business owner. Update the accepted status set only after approval, then update documentation and tests. Do not silently map it to another status.

---

# Part 17 — Revision map

| Term | Remember |
|---|---|
| Data quality | Fitness for intended business use |
| Accuracy | Correctness against a trusted reference |
| Completeness | Required data is present |
| Consistency | Related values and systems agree |
| Validity | Values follow agreed rules |
| Uniqueness | Business identity is not duplicated |
| Freshness | Data is recent enough |
| Quality rule | Testable expectation |
| Blocking test | Prevents unsafe publication |
| Warning test | Reports an issue without blocking |
| Quarantine | Preserves rejected records for investigation |
| Quality gate | Decision point before data moves forward |
| Reconciliation | Compare source and target evidence |
| Metric contract | Documented business definition |

## Final competency checklist

- [ ] Explain the six quality dimensions.
- [ ] Convert business requirements into testable rules.
- [ ] Write null, duplicate, range and accepted-value checks.
- [ ] Explain why accuracy needs a trusted reference.
- [ ] Distinguish business freshness from technical freshness.
- [ ] Design reject, quarantine and block actions.
- [ ] Explain dbt data tests.
- [ ] Design a quality gate.
- [ ] Build a quality summary and interpret it.
- [ ] Explain why a successful pipeline is not proof of correct data.

**Next lesson — Day 46: Data Reconciliation and Lineage.** We will trace source records to warehouse outputs, compare counts and totals, investigate mismatches and document where every important value came from.
