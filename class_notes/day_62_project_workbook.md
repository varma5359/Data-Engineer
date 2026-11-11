# Day 62 Project Workbook — Resilient Workflow

## Goal

Design and configure a workflow that handles temporary failures, historical runs and safe retries without duplicating data.

## Scenario

TechStore processes one sales partition per day. A temporary database timeout may occur. Some source files arrive late. The company may need to reprocess a historical date after a business-rule correction.

## Step 1 — Define the business interval

Document:
- schedule and timezone;
- start date;
- data interval;
- source date column;
- output partition;
- report deadline.

Explain which date's data a run should process and why the worker's current date is not reliable for backfills.

## Step 2 — Configure a retry policy

Create a small DAG with:
- retries;
- retry_delay;
- optional exponential backoff;
- execution timeout.

Explain which failure types are temporary and which require correction rather than retry.

## Step 3 — Simulate a temporary failure

Create a controlled task that fails on its first attempt and succeeds on a later attempt, using a safe classroom-only mechanism. Do not use a real production source or create an infinite retry loop.

Record task attempts and logs.

## Step 4 — Design a permanent failure response

Simulate a validation error such as a missing required column. Explain why retrying unchanged input will not fix the problem.

## Step 5 — Design idempotent output

Choose one approach:
- replace a specific date partition safely;
- merge/upsert by a stable business key;
- use a transaction or table format with supported commit semantics.

Describe how a retry behaves after a partial write.

## Step 6 — Plan a backfill

Select a historical date range for a classroom example. Document:
- expected number of runs;
- source availability;
- output overwrite/merge behavior;
- resource limits;
- validation;
- notification plan.

Do not start a large backfill in a shared environment.

## Step 7 — Capacity controls

Identify one shared source system and define a safe concurrency limit. Explain how an Airflow pool can protect it.

## Step 8 — Submission

Submit:
1. schedule/timezone design;
2. data interval explanation;
3. retry configuration;
4. temporary failure evidence;
5. permanent failure analysis;
6. idempotency design;
7. backfill plan;
8. pool/concurrency decision;
9. README with environment/version and validation status.

## Completion criteria

A reviewer can explain what period the DAG processes, how temporary failures are retried, how permanent failures are surfaced, how historical data is reprocessed and how reruns avoid duplicate output.
