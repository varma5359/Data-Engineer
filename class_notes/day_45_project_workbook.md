# Day 45 Project Workbook — Data Quality Framework

## Goal

Create a small quality rule set for an order dataset and produce evidence for each check.

## Step 1 — Confirm the contract

Document:
- table grain;
- primary/business key;
- required fields;
- allowed status values;
- amount rules;
- freshness expectation;
- owner and severity.

## Step 2 — Create the training table

Use the Day 45 class-note SQL fixture.

## Step 3 — Implement checks

Run:
- required order ID;
- required customer ID according to the training rule;
- duplicate order ID;
- allowed status;
- positive paid amount;
- future-date rule if approved.

## Step 4 — Create a quality summary

Record total rows and failed counts for each rule.

## Step 5 — Decide severity

Classify each rule as:
- blocking;
- warning;
- informational.

Explain the business reason.

## Step 6 — Rejected-record handling

For each failed row, record:
- source identity;
- failed field;
- safe rejected value;
- rule ID;
- reason;
- run ID.

Do not silently delete records or replace invalid amounts with zero.

## Step 7 — Failure drills

Explain:
1. A source status value is newly introduced.
2. The source feed is stale but the pipeline succeeded.
3. Counts match but duplicate keys exist.
4. An optional field is missing for 2% of rows.
5. A critical reconciliation check fails.

## Submission

- quality rule catalogue;
- SQL checks;
- outputs;
- severity policy;
- rejected-record example;
- README describing the quality gate.
