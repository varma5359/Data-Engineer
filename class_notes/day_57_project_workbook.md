# Day 57 Project Workbook — Distributed Sales Analysis

## Goal

Join orders with customer data, identify unmatched records, validate join cardinality and experiment with Spark partitions.

## Step 1 — Create the practice DataFrames

Use the orders and customers sample data from the class notes. Record row counts and distinct customer keys.

## Step 2 — Run join types

Run:
- inner;
- left;
- right;
- full outer;
- left semi;
- left anti.

For each, record:
- join type;
- output row count;
- which keys remain;
- whether null values appear;
- business use case.

## Step 3 — Add product data

Create a products DataFrame with product_id, product_name and category. Extend orders with product_id and join the datasets. Use a valid business key and select explicit output columns.

## Step 4 — Investigate duplicate dimension keys

Create two customer rows with the same customer_id. Join them to orders. Record how many rows appear for the duplicated key and explain why totals can inflate.

## Step 5 — Check unmatched keys

Use left_anti to find orders without customers. Also check customers without orders by reversing the left and right DataFrames or using a full outer join.

## Step 6 — Null-key review

Add one order with a null customer_id. Observe standard equality join behavior. Document a business rule for missing customer keys rather than replacing all nulls with a shared placeholder automatically.

## Step 7 — Partition experiment

Record the initial partition count. Run repartition(4), then coalesce(2). Record counts and explain why repartition generally shuffles while coalesce is used to reduce partitions.

## Step 8 — Write partitioned output

Write orders partitioned by city. Inspect the output directory. Explain why high-cardinality fields may create too many small partitions.

## Step 9 — Validate

Compare:
- source order count;
- join row count;
- distinct order IDs;
- total amount before and after join;
- unmatched key count;
- duplicate dimension key count.

Explain every difference.

## Step 10 — Submission

Submit:
1. code/notebook;
2. join result samples;
3. join comparison table;
4. unmatched records;
5. duplicate-key analysis;
6. partition count evidence;
7. partitioned output folder view;
8. row-count and total validation;
9. README with environment details.

## Completion criteria

You can choose a join type based on a business requirement, detect missing or duplicate keys, explain row multiplication and choose a partition strategy without assuming more partitions are always better.
