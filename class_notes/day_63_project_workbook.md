# Day 63 Project Workbook — Automated Quality Gate

## Goal

Build an automated validation process for TechStore orders and connect its pass/fail result to an Airflow workflow.

## Input data

Use the six sample orders from the Day 63 class notes. The dataset intentionally contains a missing customer ID, negative amount, duplicate order ID and invalid status.

## Step 1 — Write the data contract

Document:
- row grain;
- required columns;
- data types;
- nullability;
- unique key;
- allowed status values;
- amount rules;
- source owner;
- accepted/rejected behavior.

Do not assume order_id is unique until you define whether the dataset contains one row per order or one row per order line.

## Step 2 — Implement individual checks

Create simple Python checks for:
1. required values;
2. negative amounts;
3. duplicate business keys;
4. allowed status values;
5. row count reconciliation.

Record failed row numbers and rule IDs. Avoid printing sensitive full records in operational logs.

## Step 3 — Build accepted and rejected outputs

Use the defined rule policy. Each input row should be accounted for exactly once in the classroom example.

Expected accounting:
- input rows: 6;
- accepted rows: 2;
- rejected rows: 4.

Explain why the duplicate policy keeps the first occurrence and rejects the second in this exercise. State that production behavior must be agreed with the source owner.

## Step 4 — Add reconciliation

Confirm:
input_count = accepted_count + rejected_count

Also compare the accepted amount total with the expected sample total. Use Decimal for financial calculations in a production-quality extension.

## Step 5 — Add an Airflow quality gate

Create tasks:
- transform_data;
- validate_quality;
- publish_data.

The validation task must raise an error when a critical check fails. Publication must depend on validation success.

Pass only small metadata between tasks.

## Step 6 — Test pass and fail paths

Test:
- a valid dataset;
- a dataset with a missing required value;
- a duplicate key;
- a negative amount;
- an invalid status;
- a count mismatch.

Record which cases warn, quarantine or fail the workflow according to your contract.

## Step 7 — Evidence

Submit:
1. data contract;
2. validation code;
3. accepted/rejected record counts;
4. failed rule report;
5. reconciliation output;
6. DAG diagram;
7. successful quality-gate run;
8. failed quality-gate run with publication blocked;
9. README with assumptions and validation status.

## Completion criteria

A reviewer can see the exact rules, understand why each row passed or failed, verify count reconciliation and confirm that critical quality failures prevent curated publication.
