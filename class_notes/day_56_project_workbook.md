# Day 56 Project Workbook — Spark Transformation Lab

## Goal

Transform the TechStore orders dataset using Spark DataFrame operations and produce a city-level sales summary.

## Step 1 — Prepare the environment

Use the Spark environment from Day 55. Read orders.csv into a DataFrame and confirm the row count is five.

## Step 2 — Column selection

Select order_id and amount. Record the output and explain why select does not remove duplicate rows.

## Step 3 — Filtering

Create:
- orders above 300;
- orders from Chennai;
- Hyderabad orders above 150;
- orders with amount between 200 and 500.

Record actual rows for each filter.

## Step 4 — Create derived columns

Add:
- source_system with a literal value;
- service_fee using the teaching rule of 10%;
- order_category using the thresholds in the notes.

Document that these are practice rules, not production tax or fee policy.

## Step 5 — Data type handling

Cast order_id to string in a new DataFrame. Inspect the schema. Explain why money fields often require DecimalType.

## Step 6 — Null handling

Create the sample DataFrame with missing customer_id and amount values. Run isNull, isNotNull, fillna and dropna. Explain why replacing missing amount with zero needs business approval.

## Step 7 — Duplicate handling

Create duplicate rows. Compare distinct() and dropDuplicates(["order_id"]). If duplicate keys contain conflicting values, propose a tie-breaking rule using an update timestamp or source priority.

## Step 8 — Aggregation

Group orders by city and calculate:
- total sales;
- average order amount;
- order count;
- minimum amount;
- maximum amount.

Record actual output.

## Step 9 — Business transformation

Filter orders where amount is at least 200. Calculate city-level order count, total sales and average order value. Sort by total sales descending. Validate the expected totals against the source rows.

## Step 10 — Write outputs

Write CSV and Parquet outputs to separate practice directories. Inspect the generated output directory and explain why Spark may create multiple part files.

## Step 11 — Submission

Submit:
- code/notebook;
- outputs for each operation;
- schema evidence;
- null and duplicate handling explanation;
- city summary;
- CSV and Parquet output evidence;
- correctness validation;
- README with environment details.

## Completion criteria

You can explain every operation, its output and whether it is a transformation or action. You can defend null, duplicate and overwrite decisions using business rules.
