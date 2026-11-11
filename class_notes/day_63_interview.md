# Day 63 Interview Guide — Automated Data Quality

## 1. What is data quality?

Data quality describes how well data meets defined requirements for its intended business use. It includes completeness, validity, uniqueness, consistency, accuracy and timeliness.

## 2. What is completeness?

Completeness checks whether required information is present. For example, every accepted order must have an order_id and customer_id. Completeness does not prove the values are factually correct.

## 3. What is validity?

Validity checks whether values follow defined rules. Examples include a non-negative order amount, a valid date format or an approved order status.

## 4. What is uniqueness?

Uniqueness checks whether a key is unique at the intended table grain. For a one-row-per-order table, order_id may be unique. For an order-line table, order_id can repeat and a line-level key is needed.

## 5. Why is grain important before duplicate checks?

Grain defines what one row represents. If a table contains multiple order lines per order, repeated order IDs are valid. A duplicate rule must use the correct business key for that grain.

## 6. What is a data contract?

A data contract documents the expectations between producers and consumers, including schema, types, required fields, keys, allowed values, freshness, ownership and change rules.

## 7. What is the difference between a schema check and a business-rule check?

A schema check validates structure such as column names and types. A business-rule check validates meaning, such as amount not being negative or status being one of the approved values.

## 8. What is a quarantine area?

It is a controlled location for rejected records or failed batches. It allows investigation and possible correction without silently mixing invalid data into the curated dataset.

## 9. Should every invalid row fail the entire pipeline?

Not always. The policy depends on rule severity and business requirements. Critical failures may stop publication. Other issues may be quarantined or recorded as warnings. The policy must be explicit and approved.

## 10. What is a quality gate?

A quality gate is a workflow step that determines whether data can move to the next stage. If critical validation fails, downstream publication should not execute.

## 11. Why is row-count reconciliation useful?

It verifies that records are accounted for between input, accepted and rejected datasets. For a process where each row is assigned exactly once, input count should equal accepted count plus rejected count. It does not alone prove data accuracy.

## 12. What is referential integrity?

It checks whether references in one dataset exist in another. For example, each order.customer_id should match a customer record, subject to documented handling for late or missing reference data.

## 13. What is the difference between warning and failure?

A warning records an issue that may not block processing, such as an unusual but possible volume change. A failure blocks a critical step, such as a missing required key. Severity thresholds should be agreed with business owners.

## 14. Why should validation results be stored?

They provide an audit trail showing which dataset and run were checked, which rules ran, how many records failed and whether the gate passed. This helps troubleshoot and measure quality trends.

## 15. Why should logs avoid full rejected records?

Records may contain personal or confidential information. Logs should contain safe identifiers, rule IDs and controlled references to quarantine storage rather than exposing sensitive values.

## 16. Scenario: row count matches yesterday but revenue is much lower.

Row count alone is insufficient. I would compare amount totals, nulls, filters, product/customer joins, refunds, source completeness and business-date boundaries. I would stop publication if the revenue reconciliation rule is critical.

## 17. Scenario: duplicate order IDs appear.

I would first confirm the dataset grain. If one row per order is expected, I would identify duplicate keys and determine the approved resolution rule, such as reject, latest-record selection or source correction. I would not arbitrarily delete rows.

## 18. Scenario: a quality check fails every day.

I would inspect the failed rule, trend and source changes. The rule may reveal a real upstream defect or may be incorrectly defined. I would work with the data owner to correct the cause or formally change the contract, not silently disable the check.

## 19. How do you integrate quality checks with Airflow?

I create validation tasks after transformation or loading and make publication depend on their success. The validation task raises a failure for critical rules and records safe metrics and rule results. Large data stays in storage; task metadata contains small summaries.

## 20. Explain your Day 63 project.

I defined a data contract for orders, checked required fields, ranges, duplicate keys and allowed statuses, separated accepted and rejected rows and reconciled counts. I added an Airflow quality task that blocks publication when a critical validation fails. I recorded rule outcomes and documented the duplicate policy and dataset grain.

## Common interview traps

- Checking only row counts.
- Assuming repeated order IDs are always duplicates.
- Treating every negative amount as invalid without business context.
- Dropping bad records silently.
- Using warnings for critical failures.
- Logging sensitive rejected records.
- Treating task success as proof of data quality.
- Disabling checks to make a workflow pass.

## Readiness checklist

- [ ] Explain quality dimensions and data contracts.
- [ ] Define grain and business keys.
- [ ] Explain schema versus business checks.
- [ ] Explain accepted/rejected/quarantine handling.
- [ ] Explain reconciliation.
- [ ] Explain quality gates and Airflow dependencies.
- [ ] Describe safe validation logging.
