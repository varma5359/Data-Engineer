# Day 49 Project Workbook — Warehouse Performance Audit

## Goal

Perform a small, evidence-based performance investigation and document trade-offs.

## Step 1 — Record the environment

Document:
- PostgreSQL version;
- machine or training environment;
- table row count;
- query scope;
- indexes present;
- cache/concurrency caveats.

## Step 2 — Create the training table

Use the Day 49 class-note generate_series example to create 200,000 synthetic rows.

## Step 3 — Capture a baseline

Run EXPLAIN for the selective order_id query.

Record:
- scan method;
- estimated rows;
- estimated cost;
- filter.

If you use EXPLAIN ANALYZE, use only the read-only SELECT in the training environment and capture your actual output.

## Step 4 — Add an index

Create the order_id index and inspect the plan again.

Do not assume the optimizer must choose it. Explain the plan you actually observe.

## Step 5 — Test partition pruning

Create the date-partitioned sales table from the notes. Run EXPLAIN with a one-month date range and record which partitions are scanned.

## Step 6 — Compare materialization

Explain the difference between:
- view;
- table;
- materialized view.

Record the refresh and freshness trade-off.

## Step 7 — Validate correctness

Compare row count and returned values before and after optimization. Performance improvement is not acceptable if the result changes incorrectly.

## Step 8 — Cost and maintenance review

For every proposed index, partition or materialized object, document:
- query benefit;
- storage cost;
- write/refresh cost;
- maintenance;
- freshness impact.

## Submission

- baseline plan;
- after-index plan;
- partition plan;
- materialization comparison;
- correctness evidence;
- optimization decision record;
- README with actual measurements and limitations.

Never invent benchmark timings.
