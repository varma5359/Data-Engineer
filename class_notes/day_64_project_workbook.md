# Day 64 Project Workbook — Pipeline Monitoring Dashboard

## Goal
Design an operational dashboard and alert policy for the TechStore daily sales pipeline.

## Scenario
The report must be available by 7:00 AM. The support team needs to know whether the pipeline ran, whether data is fresh, how many rows were processed and whether quality checks passed.

## Step 1 — Define service expectations
Document the expected schedule, report deadline, freshness target, source arrival, normal duration (measured or clearly labeled assumed), critical quality checks, owner and escalation contact.

## Step 2 — Select metrics
Include:
- latest DAG run status;
- last successful data interval;
- freshness;
- task duration;
- input and output rows;
- accepted/rejected counts;
- retry and failure counts;
- quality-gate status.

For each metric, define its meaning, source and business value.

## Step 3 — Design dashboard sections
Create a wireframe containing:
1. overall pipeline health;
2. latest run and data interval;
3. freshness and deadline status;
4. task duration and retries;
5. input/output volume;
6. quality results;
7. incident history.

## Step 4 — Define alert rules
Write alert conditions for DAG failure, stale report, missing source file, rejected records above an approved threshold, unusual volume and excessive task duration. Define severity, owner, channel and first action.

## Step 5 — Write a runbook
Document investigation and recovery for a failed sales pipeline. Include a warning not to rerun a partially committed non-idempotent load blindly.

## Step 6 — Add safe logs
Create a small Python logging example with INFO, WARNING and ERROR. Include pipeline/run identifiers but no secrets or customer details.

## Step 7 — Simulate an incident
Choose missing source data or failed validation. Record detection time, alert content, diagnostic evidence, mitigation, recovery verification and prevention action.

## Step 8 — Submission
Submit service expectations, metric dictionary, dashboard wireframe, alert policy, sample logs, runbook, incident timeline and README with assumptions/validation status.

## Completion criteria
An operator can answer whether the pipeline is healthy, whether data is fresh, what failed and who should respond.
