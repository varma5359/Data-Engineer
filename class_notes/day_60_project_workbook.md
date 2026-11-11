# Day 60 Project Workbook — Workflow DAG Design

## Project goal

Design a valid Airflow DAG for a daily retail sales workflow. This is a design and reasoning exercise. Day 61 introduces runnable DAG code.

## Business scenario

TechStore must publish yesterday's sales report by 7:00 AM. The pipeline receives an order file, validates it, transforms records, loads the warehouse, checks totals and publishes the report.

## Step 1 — Write the workflow requirement

Record:
- business owner;
- report deadline;
- source file location;
- expected arrival time;
- required columns;
- business key;
- output table or location;
- quality rules;
- notification owner.

Mark unknown volumes and times as assumptions.

## Step 2 — Break the process into tasks

Use these starting tasks:
1. check_source;
2. validate_source;
3. transform_orders;
4. load_warehouse;
5. quality_check;
6. publish_report.

For each task, write:
- purpose;
- input;
- output;
- failure condition;
- whether it is safe to retry;
- expected owner.

## Step 3 — Draw the DAG

Draw the task dependencies. Start with a sequential chain. Then identify whether any tasks can safely run in parallel.

Check that:
- every task has a clear upstream/downstream relationship;
- no circular dependency exists;
- report publication happens only after quality checks succeed.

## Step 4 — Simulate a failure

Describe what happens if:
- source file is missing;
- schema validation fails;
- transformation fails;
- warehouse load partially writes;
- quality check detects a mismatch;
- report publication fails.

For each case, decide which tasks must not run and what should be rerun.

## Step 5 — Design safe retries

Identify non-idempotent operations. For each one, choose a safe strategy such as deterministic partition replacement, merge/upsert by a stable key or transactional commit.

Explain why a retry must not duplicate yesterday's sales.

## Step 6 — Map the Airflow architecture

Draw and label:
- DAG files;
- DAG processor;
- scheduler;
- executor;
- worker/process;
- metadata database;
- web interface;
- triggerer (conceptual).

Write one sentence describing each component.

## Step 7 — Decide what data is passed between tasks

List examples of small metadata:
- run ID;
- source path;
- output path;
- row count;
- validation status.

List large datasets that should remain in storage:
- full orders DataFrame;
- raw CSV contents;
- Parquet files.

## Step 8 — Submission

Submit:
1. workflow requirements;
2. task responsibility table;
3. DAG diagram;
4. failure and recovery table;
5. idempotency decisions;
6. Airflow architecture diagram;
7. small-metadata versus large-data handoff explanation;
8. README.

## Completion criteria

A reviewer can understand what the workflow does, why tasks run in that order, what happens when a task fails and how the pipeline avoids publishing incomplete or duplicate data.
