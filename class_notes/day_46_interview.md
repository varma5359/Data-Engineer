# Day 46 Interview Guide — Data Reconciliation and Lineage

## 1. What is reconciliation?

Reconciliation compares related source and target data to verify that expected records and values were transferred or transformed correctly. It can include row counts, distinct keys, missing and extra keys, aggregate totals, grouped totals and field-level comparisons.

## 2. Why are row counts not enough?

The source and target can have the same number of rows while containing different records. One row may be missing and another duplicated. Counts must be combined with key, value and aggregate checks.

## 3. Why are matching totals not enough?

Different errors can offset each other. One amount can be too high and another too low while the grand total remains unchanged. I would compare keys, grouped totals and individual values for suspicious records.

## 4. What is source-to-target mapping?

It documents how source fields map to target fields, including renames, type conversions, filters, business rules, null behavior and key relationships. It makes transformations understandable and supports change impact analysis.

## 5. What is a control total?

A control total is a recorded count or amount used to compare processing boundaries, such as source rows, accepted rows, rejected rows and target rows. Its scope and calculation must be documented.

## 6. What is data lineage?

Lineage describes where data originated, how it moved and which transformations produced the final data. It supports debugging, audit, impact analysis and trust.

## 7. Technical versus business lineage?

Technical lineage traces systems, files, tables, columns, jobs and model dependencies. Business lineage explains the meaning and rules behind a metric, such as what counts as paid sales.

## 8. What is column-level lineage?

It traces an output column to the upstream columns and transformations that produced it. For example, line_sales_amount may be derived from quantity multiplied by unit_price.

## 9. How do you investigate a source-target mismatch?

I confirm the scope and filters, compare counts, distinct keys, missing and extra keys, totals and grouped totals, then compare values for mismatched keys. I inspect transformations, late arrivals and duplicates, document the root cause and rerun validation after correction.

## 10. What metadata should a load record?

Run ID, source system, source batch or file, extraction time, load time, watermark, row counts, accepted/rejected counts, model version, status and reconciliation result.

## 11. What is impact analysis?

Impact analysis identifies downstream tables, columns, dashboards or processes affected by a source or model change. Lineage helps identify the affected path before changes are deployed.

## 12. Scenario: source and target counts match but totals differ.

I would compare aggregate values by group, inspect field mappings and conversions, and find records where values differ by business key. The row count only proves equal cardinality, not equal content.

## 13. Scenario: totals match but keys differ.

I would compare source-only and target-only keys using anti-joins or set comparisons. Matching totals can hide missing and extra records. I would not approve the load until the key difference is explained.

## 14. Scenario: a source column changes name.

I would use source-to-target mapping and lineage to identify dependent staging models, facts, dimensions and reports. I would coordinate the schema change, update ingestion and transformations, run tests and communicate downstream impact.

## 15. Explain your Day 46 project.

I created source and target training tables, defined the eligible load scope, compared counts and keys, checked missing and extra records, compared totals by date, injected a controlled amount mismatch and identified it through field-level comparison. I documented source-to-target mapping and lineage.

## Interview traps

- Treating equal row counts as proof of equality.
- Comparing totals with different filters or time zones.
- Ignoring grouped reconciliation.
- Failing to preserve run metadata.
- Calling a table dependency graph complete business lineage.
- Fixing target data manually without evidence or retesting.

## Readiness checklist

- [ ] Explain count, key, aggregate and field reconciliation.
- [ ] Explain why totals can hide errors.
- [ ] Describe source-to-target mapping.
- [ ] Explain technical and business lineage.
- [ ] Defend the mismatch investigation in your project.
