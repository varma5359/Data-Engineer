# Day 39: Pipeline Logging and Observability

**Phase:** 4 — Data Processing and Pipelines  
**Syllabus topic:** Pipeline Logging and Observability  
**Mini-project:** Pipeline Observability Tool  
**Learning time:** 2–3 hours  
**Level:** Complete beginner to junior Data Engineer  

---

# 1. What are we learning today?

A job can exit zero but load zero rows or run slowly. Operators need evidence beyond a success message.

## 1.1 Learning outcomes

By the end of this lesson, you should be able to:

1. Define operator questions.
2. Generate and propagate run ID.
3. Log stage start/end, source, counts, duration.
4. Capture error context without secrets.
5. Measure rows/files, rejects, retries, lag and duration.
6. Define thresholds/alerts.
7. Compare runs in dashboards.
8. Write recovery runbooks.
9. Test by deliberately failing a run.

## 1.2 Pipeline context

```text
Business source -> Pipeline Logging and Observability -> Validate/process -> Staging/destination -> Reconcile -> Checkpoint + monitoring
```

A pipeline is not merely a script that reads data. It is repeatable work with defined input, rules, output, progress, error path and evidence of correctness.

# 2. Business scenario and vocabulary

A job can exit zero but load zero rows or run slowly. Operators need evidence beyond a success message.

| Term | Beginner meaning | Why it matters |
|---|---|---|
| **Log** |  execution event | Used to design and explain reliable pipelines. |
| **metric** |  numeric measurement | Used to design and explain reliable pipelines. |
| **trace** |  connected work | Used to design and explain reliable pipelines. |
| **run ID** |  execution identifier | Used to design and explain reliable pipelines. |
| **structured log** |  named fields | Used to design and explain reliable pipelines. |
| **SLA** |  service agreement | Used to design and explain reliable pipelines. |
| **SLO** |  target | Used to design and explain reliable pipelines. |
| **alert** |  notification | Used to design and explain reliable pipelines. |
| **observability** |  infer state from outputs. | Used to design and explain reliable pipelines. |

## Before coding, answer five questions

1. Where does data come from and who owns it?
2. Which records/fields must be collected or changed?
3. What can go wrong (missing values, duplicates, timeout, schema change, partial failure)?
4. Where does the result go and what is its schema?
5. How will correctness be proven (counts, keys, totals, status and run metadata)?

# 3. Understand the concept before coding

## 3.1 A small record example

| ID | Value | Status |
|---:|---:|---|
| 1 | 100 | valid |
| 2 | 200 | valid |
| 3 | -5 | invalid |

A reliable run accounts for all 3 input records: 2 accepted, 1 rejected with a reason, 2 output records. It must not silently discard record 3.

## 3.2 Flow to remember

```text
Receive -> Inspect -> Validate -> Process -> Persist -> Verify -> Record progress
                    |                  |
                    v                  v
                Reject path       Retry/recovery path
```

- Receive: obtain source data.
- Inspect: understand shape, size and metadata.
- Validate: check rules before trusting it.
- Process: apply topic-specific operation.
- Persist: write durably.
- Verify: compare expected and actual results.
- Record progress: checkpoint only when safe.

# 4. First practical example

## Task

Demonstrate structured logs with one small operation before scaling it into a pipeline.

## Code

```python
import logging
logging.basicConfig(level=logging.INFO, format="%(levelname)s %(message)s")
logging.info("run_id=%s rows_read=%d rows_written=%d status=%s",
             "run-001", 12000, 11998, "SUCCESS")
```

## Expected output

```text
INFO run_id=run-001 rows_read=12000 rows_written=11998 status=SUCCESS
```

> Sample output is for the stated sample values. API/database/file results may vary. Do not claim variable outputs are guaranteed or runtime-tested unless you executed them.

## Line-by-line explanation

logging.info emits INFO. Include run ID/counters. Production JSON logging stores fields separately; simple format demonstrates concept.

### After running, answer

- What input did I provide?
- Which operation ran?
- Which object holds the result?
- What does output prove?
- What does output not prove?

A successful request/query/read does not automatically prove completeness, uniqueness, business validity or correctness.

# 5. Build the pipeline step by step

## Step 1 — Define the input contract

Document source and owner, fields/types, required/optional values, unique identity, time/batch boundary, expected volume/frequency, permissions, and source limitations. If these are unknown, clarify before coding.

## Step 2 — Read a tiny sample

Start with 3–10 safe records. Inspect field names, types, nulls, duplicate keys, cursor/timestamp/version and whether the sample represents real edge cases. Never print credentials or sensitive data.

## Step 3A — Validation is a decision

| Rule | Example | Violation action |
|---|---|---|
| Required field | ID exists | Reject/quarantine |
| Type | Amount numeric | Record conversion reason |
| Range | Quantity positive | Reject or business review |
| Identity | Key unique | Deduplicate or fail per contract |
| Schema | Required columns exist | Stop batch and alert |
| Completeness | All pages/partitions processed | Mark incomplete; don't checkpoint |

## Step 4 — Process bounded units

A unit can be a page, file, fetch batch, partition or record group. Bounded units control memory, reduce retry scope, provide progress points and isolate failures. Select the unit based on source/target capacity—not a tutorial's arbitrary number.

## Step 5 — Persist safely

Ask: did destination commit finish? Can the batch replay without duplicate effects? Is there a unique key/idempotency token? Can I trace output to source? What if process crashes now?

## Step 6 — Verify

Use at least two: input = accepted + rejected; key uniqueness; totals/ranges; boundary gap/overlap; schema; freshness; source/target reconciliation.

# 6. Mistakes and corrections

## Mistake 1: Log secrets/PII

**Why dangerous:** Security risk.

**Correct approach:** Redact/minimize.

## Mistake 2: Only log success

**Why dangerous:** Cannot diagnose zero rows/slow run.

**Correct approach:** Include counts/duration/reconciliation.

## Mistake 3: Alert every error

**Why dangerous:** Alert fatigue.

**Correct approach:** Threshold, dedupe and severity.

## Mistake 4: Customer ID metric labels

**Why dangerous:** High cardinality.

**Correct approach:** Bound metric labels; details in logs.

## Mistake 5: No run ID

**Why dangerous:** Concurrent events cannot correlate.

**Correct approach:** Propagate run ID.


## Debugging checklist

1. Confirm source exists and request/query is correct.
2. Inspect exact raw input for a small failing case.
3. Confirm schema and types.
4. Check filters and boundaries (>, >=, dates, cursor).
5. Check mapping/business rules.
6. Check duplicate and unique-key behavior.
7. Check destination write/transaction result.
8. Inspect checkpoint and update timing.
9. Reconcile input, accepted, rejected and output counts.
10. Reproduce with controlled input before changing the full job.

# 7. Mini-project — Pipeline Observability Tool

## Objective

Build a beginner-readable pipeline demonstrating pipeline logging and observability with a happy path, invalid/failure path and verification step.

## Requirements

- Reproducible sample input and defined contract.
- Predictable processing order and bounded units.
- Required-field and identity validation.
- Accepted/rejected outcomes separated.
- Safe rerun without duplicate effects.
- Run metadata: run ID, start/end, counts and status.
- Progress/checkpoint behavior explained.
- Output reconciled.

## Sample business data

| record_id | event_time | customer_id | amount | status |
|---:|---|---:|---:|---|
| 501 | 2026-11-10 09:00:00 | 11 | 1200.00 | PAID |
| 502 | 2026-11-10 09:15:00 | 12 | 850.00 | PAID |
| 503 | 2026-11-10 10:00:00 | 11 | 499.00 | PENDING |
| 504 | 2026-11-10 10:30:00 | 13 | 0.00 | CANCELLED |

These are teaching records. A zero amount or CANCELLED status is not automatically invalid; the business contract decides.

## Part A — Design

1. Name source/destination.
2. Define unique key and required fields/types.
3. Decide batch boundary and accepted/rejected rules.
4. Define checkpoint and output schema.
5. Define reconciliation checks.

## Part B — Happy path

1. Read a small batch.
2. Count received records.
3. Validate required fields.
4. Process accepted records.
5. Persist/display output.
6. Verify counts and values.

## Part C — Failure tests

Test missing field, duplicate key and one transient error/malformed payload. Record expected behavior, actual behavior and correction. A happy-path-only script is not reliable yet.

## Part D — Replay

Run input twice; inspect duplicates; interrupt before checkpoint; restart from safe progress; reconcile final target.

## Acceptance checklist

- [ ] Input contract documented.
- [ ] Small sample reproducible.
- [ ] Code split into readable stages.
- [ ] Outputs shown after important steps.
- [ ] Invalid data has a reason.
- [ ] Retry behavior bounded.
- [ ] Rerun behavior safe.
- [ ] Checkpoint order explained.
- [ ] Counts and values reconciled.
- [ ] Limitations and production improvements documented.

# 8. Production thinking

A classroom script runs on small data. Production additionally considers scale/memory, least-privilege security, secret management, concurrency, retries/checkpoints, source load, data contracts, monitoring/alerts/runbooks, schema changes, cost and auditability. A notebook is not production-ready merely because it executes successfully.

# 9. Practice exercises

## Exercise 1 — Explain

Explain Log to a beginner; give a business example; name a failure; explain detection and recovery.

## Exercise 2 — Trace one record

Trace record 502 from source through validation, processing, destination, checkpoint and verification.

## Exercise 3 — Boundary test

Create records immediately before, exactly at and after the boundary. Explain which are processed and why.

## Exercise 4 — Failure test

Interrupt after output persistence but before checkpoint. Restart and explain idempotency and duplicate checks.

## Exercise 5 — Quality test

Make one field invalid, one key duplicated and one record late. Decide action from contract and justify.

## Exercise 6 — Interview practice

Answer aloud: problem solved; end-to-end flow; failure behavior; completeness proof; production improvements.

# 10. Day 39 revision sheet

**Flow:** Define contract -> Read bounded input -> Validate -> Process -> Persist -> Verify -> Checkpoint -> Observe

Never skip: unique record identity; what success guarantees; replay behavior; invalid versus transient error; evidence of destination correctness.

## One-minute project defence

“My Pipeline Observability Tool receives data from a defined source. It validates the contract and identity, processes controlled units, handles invalid/transient cases explicitly, persists safely, records progress after durable success and reconciles counts/values. I can rerun it safely and diagnose failures from run metadata. Before production I would add automated tests, secret management, monitoring, access controls and scale testing.”

## Self-assessment

- [ ] I can define terms without memorizing.
- [ ] I can explain code and output.
- [ ] I can trace a record end-to-end.
- [ ] I can reproduce/debug a failure.
- [ ] I can explain checkpoint ordering and replay.
- [ ] I can prove correctness with checks.
- [ ] I can defend the project.

**Complete the day when you can explain, run, interpret, test failure and prove the result—not merely read the notes.**


# 11. Mentor-led deep dive — Observability

Read each scenario slowly. Identify input, risk, control and proof.

## 11.1 Raw/curated distinction

Run lifecycle statuses RUNNING, SUCCESS, SUCCESS_WITH_REJECTS, FAILED, CANCELLED; define transitions and completion criteria.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.2 Walkthrough

Structured event fields run_id, stage, source, read/written/rejected, retries, duration, status/error code; avoid secrets.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.3 Normalization policy

Metrics: rows_read/written, rejects, duration, lag, retry/failure totals; define units and aggregation.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.4 Mapping design

Freshness: latest source event versus now/target observation; distinguish source lateness from pipeline delay.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.5 Worked business example

Alerts: no run by deadline, duration 3x baseline, unexpected zero rows, rejects threshold, lag SLO, repeated retries.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.6 Filter semantics

Alert quality: severity, owner, dedupe, actionable message/runbook; alert symptoms users care about.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.7 Aggregation grain

Dashboard: run status/duration, counts, freshness, retries/errors, source/destination health and run IDs.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.8 Transformation order

Redact tokens/passwords, minimize PII, restrict access/retention; stack traces can leak connection details.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.9 Reject evidence

Runbook explains find run ID, inspect logs/metrics, verify source/target, restart/replay, escalate and close incident.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

# 12. Guided lab checkpoints

## Lab checkpoint 1

- Predict the result before running.
- Execute this stage on a small input.
- Record output, count and error (if any).
- Explain which input/rule caused the result.
- Compare actual result with contract and expected output.
- Reproduce a failure with the smallest input and fix its cause.

## Lab checkpoint 2

- Predict the result before running.
- Execute this stage on a small input.
- Record output, count and error (if any).
- Explain which input/rule caused the result.
- Compare actual result with contract and expected output.
- Reproduce a failure with the smallest input and fix its cause.

## Lab checkpoint 3

- Predict the result before running.
- Execute this stage on a small input.
- Record output, count and error (if any).
- Explain which input/rule caused the result.
- Compare actual result with contract and expected output.
- Reproduce a failure with the smallest input and fix its cause.

## Lab checkpoint 4

- Predict the result before running.
- Execute this stage on a small input.
- Record output, count and error (if any).
- Explain which input/rule caused the result.
- Compare actual result with contract and expected output.
- Reproduce a failure with the smallest input and fix its cause.

## Lab checkpoint 5

- Predict the result before running.
- Execute this stage on a small input.
- Record output, count and error (if any).
- Explain which input/rule caused the result.
- Compare actual result with contract and expected output.
- Reproduce a failure with the smallest input and fix its cause.

## Lab checkpoint 6

- Predict the result before running.
- Execute this stage on a small input.
- Record output, count and error (if any).
- Explain which input/rule caused the result.
- Compare actual result with contract and expected output.
- Reproduce a failure with the smallest input and fix its cause.

## Lab checkpoint 7

- Predict the result before running.
- Execute this stage on a small input.
- Record output, count and error (if any).
- Explain which input/rule caused the result.
- Compare actual result with contract and expected output.
- Reproduce a failure with the smallest input and fix its cause.

## Lab checkpoint 8

- Predict the result before running.
- Execute this stage on a small input.
- Record output, count and error (if any).
- Explain which input/rule caused the result.
- Compare actual result with contract and expected output.
- Reproduce a failure with the smallest input and fix its cause.


# 13. Instructor review rubric

| Skill | Beginner evidence |
|---|---|
| Concept | Explains what/why plainly |
| Code | Reads/explains lines, not blindly copies |
| Output | Derives output from input/rule |
| Quality | Identifies invalid, duplicate and boundary cases |
| Reliability | Explains retry, replay and checkpoint |
| Verification | Uses counts/keys/totals/quality checks |
| Interview | Gives decision, reason, failure mode and proof |

