# Day 45 Interview Guide — Data Quality and Validation

## 1. What is data quality?

Data quality describes how well data meets the requirements of its intended use. It is not simply whether a file loads or a SQL query runs. A sales dashboard needs valid keys, correct business values, complete source coverage and fresh data.

## 2. Explain the main data quality dimensions.

Accuracy means values correctly represent business facts. Completeness means required data is present. Consistency means related values agree across systems and records. Validity means values follow agreed formats and rules. Uniqueness means business identities are not duplicated. Freshness means data is recent enough for its use.

## 3. What is the difference between accuracy and validity?

A value can be valid in format but inaccurate in reality. For example, a city name can be a valid string but still be the wrong city. Validity checks format and rules; accuracy requires comparison with a trusted reference or evidence.

## 4. How do you test completeness?

Identify required fields and expected records, then check for missing values and missing source deliveries. Completeness should be evaluated at field, record, table and feed levels. Optional fields should not be treated as mandatory without a business rule.

## 5. How do you test uniqueness?

Define the table grain and business key, then group by the key and find values with more than one row. The correct key depends on grain: order_id may be unique in an order-header table but not in an order-line table.

## 6. What is freshness?

Freshness measures how recently data reflects source events or updates relative to an agreed expectation. A pipeline can run successfully while loading stale source data, so monitor source timestamps and delivery completeness, not only job execution time.

## 7. What is a quality rule?

A quality rule is a testable expectation about data, such as order_id must be unique or paid order amount must be positive. A useful rule has an owner, severity, test logic and failure action.

## 8. What is a quality gate?

A quality gate is a decision point that allows, warns or blocks data from moving to the next stage based on test results. Critical failures such as duplicate business keys or failed reconciliation may block publication.

## 9. What is quarantine?

Quarantine preserves records that fail validation in a separate controlled area with their source identity and rejection reason. It allows investigation and correction without silently discarding data.

## 10. Why should invalid amounts not be replaced with zero?

Zero is a valid business value and has a different meaning from an unknown or invalid amount. Replacing bad data with zero can distort financial metrics. Preserve the original value, record the reason and apply only an approved correction.

## 11. What is the difference between a warning and a blocking failure?

A warning reports an issue that may be acceptable under an agreed policy. A blocking failure prevents publication because the data is unsafe. Severity should reflect business impact and be approved by the data owner.

## 12. What does a passing data test prove?

It proves that the tested condition passed for the evaluated data. It does not prove all business definitions are correct or that every possible defect is absent. Tests cover explicit expectations only.

## 13. How do dbt tests help?

dbt can run tests such as not-null, unique, accepted-values and relationship checks on models. Custom SQL tests can represent business rules. Test syntax depends on the dbt version, and the team must still decide severity and failure behavior.

## 14. Scenario: job succeeded but dashboard has zero rows.

I would inspect source freshness and completeness, extraction filters, pagination, validation acceptance counts, destination writes and reconciliation. A successful process status is not enough. The pipeline should detect unexpected zero rows when the business contract expects data.

## 15. Scenario: total amount matches but duplicate and missing records exist.

The dataset is not necessarily correct. Two errors can offset each other in an aggregate. I would compare keys, counts, samples, business totals and source boundaries rather than relying on one total.

## 16. Scenario: a new status appears.

I would inspect the source contract and consult the business owner. If the new status is valid, I would update the mapping, documentation and accepted-values test. I would not silently map it to an unrelated status.

## 17. How would you design quality checks for a sales mart?

I would test required and unique fact keys, dimension relationships, valid statuses, positive or otherwise approved amounts, date ranges, source completeness, freshness and reconciliation. I would classify checks by severity and block publication when critical rules fail.

## 18. Explain your Day 45 quality project.

I defined the dataset grain and rules, created SQL checks for missing values, duplicate keys, allowed statuses and paid amount validity, summarized failures and documented warning versus blocking actions. I preserved rejected-record context rather than silently deleting rows.

## Interview traps

- Equating validity with accuracy.
- Checking duplicates without declaring grain.
- Treating every null as an error.
- Using only row counts or totals to claim correctness.
- Treating pipeline success as data freshness.
- Silently replacing invalid data.
- Making every check warning or every check blocking.

## Readiness checklist

- [ ] Explain the quality dimensions.
- [ ] Give one test for each dimension.
- [ ] Explain severity and quality gates.
- [ ] Describe quarantine and correction.
- [ ] Defend the quality checks in your project.
