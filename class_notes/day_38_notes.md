# Day 38: Pipeline Reliability and Failure Handling

**Phase:** 4 — Data Processing and Pipelines  
**Syllabus topic:** Pipeline Reliability and Failure Handling  
**Mini-project:** Reliable Pipeline Simulator  
**Learning time:** 2–3 hours  
**Level:** Complete beginner to junior Data Engineer  

---

# 1. What are we learning today?

Pipeline may crash after reading, partially writing or committing before checkpoint. Design replay to preserve correctness.

## 1.1 Learning outcomes

By the end of this lesson, you should be able to:

1. Map failure points.
2. Classify transient/permanent errors.
3. Set finite retries/backoff/jitter/timeouts.
4. Use stable keys and idempotent writes.
5. Persist progress after durable output.
6. Use transactions or temp-file atomic publication.
7. Isolate bad records with metadata.
8. Make replay explicit and safe.
9. Test crash points and reconcile.

## 1.2 Pipeline context

```text
Business source -> Pipeline Reliability and Failure Handling -> Validate/process -> Staging/destination -> Reconcile -> Checkpoint + monitoring
```

A pipeline is not merely a script that reads data. It is repeatable work with defined input, rules, output, progress, error path and evidence of correctness.

# 2. Business scenario and vocabulary

Pipeline may crash after reading, partially writing or committing before checkpoint. Design replay to preserve correctness.

| Term | Beginner meaning | Why it matters |
|---|---|---|
| **Transient** |  temporary | Used to design and explain reliable pipelines. |
| **permanent** |  requires correction | Used to design and explain reliable pipelines. |
| **retry policy** |  attempts/delay | Used to design and explain reliable pipelines. |
| **exponential backoff** |  growing wait | Used to design and explain reliable pipelines. |
| **idempotent** |  safe repeat | Used to design and explain reliable pipelines. |
| **dead-letter** |  isolated failures | Used to design and explain reliable pipelines. |
| **atomicity** |  all-or-nothing | Used to design and explain reliable pipelines. |
| **recovery** |  return to known correct state. | Used to design and explain reliable pipelines. |

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

Demonstrate idempotency with one small operation before scaling it into a pipeline.

## Code

```python
import time
for attempt in range(1, 4):
    try:
        print("Attempt", attempt)
        # call the operation here
        break
    except TimeoutError:
        if attempt == 3:
            raise
        time.sleep(2 ** attempt)
```

## Expected output

```text
Attempt 1
Attempt 2
Attempt 3
```

> Sample output is for the stated sample values. API/database/file results may vary. Do not claim variable outputs are guaranteed or runtime-tested unless you executed them.

## Line-by-line explanation

At most three attempts; waits 2 then 4 seconds. Production logic classifies errors, caps delay, adds jitter and avoids retrying non-idempotent effects blindly.

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

## Mistake 1: Retry forever

**Why dangerous:** Hides outage/consumes resources.

**Correct approach:** Bound attempts and alert.

## Mistake 2: Retry POST blindly

**Why dangerous:** May duplicate effects.

**Correct approach:** Idempotency key/reconcile.

## Mistake 3: Catch all and continue

**Why dangerous:** Hides defects/data loss.

**Correct approach:** Catch expected errors; surface others.

## Mistake 4: Claim exactly once casually

**Why dangerous:** Failures cause replay.

**Correct approach:** At-least-once + idempotent effect; state guarantee.

## Mistake 5: Delete raw before output durable

**Why dangerous:** Loses evidence.

**Correct approach:** Retain until verified and retention allows.


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

# 7. Mini-project — Reliable Pipeline Simulator

## Objective

Build a beginner-readable pipeline demonstrating pipeline reliability and failure handling with a happy path, invalid/failure path and verification step.

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

Explain Transient to a beginner; give a business example; name a failure; explain detection and recovery.

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

# 10. Day 38 revision sheet

**Flow:** Define contract -> Read bounded input -> Validate -> Process -> Persist -> Verify -> Checkpoint -> Observe

Never skip: unique record identity; what success guarantees; replay behavior; invalid versus transient error; evidence of destination correctness.

## One-minute project defence

“My Reliable Pipeline Simulator receives data from a defined source. It validates the contract and identity, processes controlled units, handles invalid/transient cases explicitly, persists safely, records progress after durable success and reconciles counts/values. I can rerun it safely and diagnose failures from run metadata. Before production I would add automated tests, secret management, monitoring, access controls and scale testing.”

## Self-assessment

- [ ] I can define terms without memorizing.
- [ ] I can explain code and output.
- [ ] I can trace a record end-to-end.
- [ ] I can reproduce/debug a failure.
- [ ] I can explain checkpoint ordering and replay.
- [ ] I can prove correctness with checks.
- [ ] I can defend the project.

**Complete the day when you can explain, run, interpret, test failure and prove the result—not merely read the notes.**


# 11. Mentor-led deep dive — Reliability

Read each scenario slowly. Identify input, risk, control and proof.

## 11.1 Raw/curated distinction

Failure map: before read, after read, transform, during write, after commit/before checkpoint, after checkpoint. For each identify durable state and restart action.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.2 Walkthrough

Retry table: timeout retry; 429 respect wait; 503 backoff; invalid schema quarantine/alert; auth stop and fix.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.3 Normalization policy

Backoff attempts 1/2/3 waits 2/4 seconds; cap and jitter to avoid synchronized retries.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.4 Mapping design

File output: temporary file -> close/validate -> atomic rename if filesystem supports; rerun checks source/run identity.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.5 Worked business example

Database output: unique key/upsert or transactionally deduplicated staging; replay must not add rows twice.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.6 Filter semantics

Poison row: isolate with reason/source offset if safe; systemic schema/logic errors should fail/alert, not be skipped.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.7 Aggregation grain

Exactly-once nuance: delivery may be at-least-once; idempotency/transaction makes effect effectively once. Explain boundaries.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.8 Transformation order

Runbook: detect -> inspect run/checkpoint -> inspect partial writes -> fix -> replay bounded input -> reconcile -> mark recovered/notify.

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



# 14. Additional worked lab cells — Reliability

Each cell is a separate experiment. Students should predict, execute, record output and explain the business meaning.

## Lab cell 1

Failure matrix: crash before read (nothing), after read (re-read), during transform (reprocess batch), during write (transaction/temp output), after commit before checkpoint (replay idempotently).

**Write in your notebook:** input; expected output; actual output; why it happened; validation evidence; what you would do if it failed.

## Lab cell 2

Retry decision table: network timeout=bounded retry; rate limit=wait; bad schema=quarantine/alert; invalid credentials=stop; destination constraint=investigate data/code.

**Write in your notebook:** input; expected output; actual output; why it happened; validation evidence; what you would do if it failed.

## Lab cell 3

Retry timing example 1,2,4,8 seconds capped; add random jitter and global retry budget. Track attempts and elapsed time.

**Write in your notebook:** input; expected output; actual output; why it happened; validation evidence; what you would do if it failed.

## Lab cell 4

Atomic file pattern: write temporary path, flush/close, validate row count, rename to final; clean abandoned temp files on restart.

**Write in your notebook:** input; expected output; actual output; why it happened; validation evidence; what you would do if it failed.

## Lab cell 5

DB pattern: stage rows, validate, transactionally upsert and commit; checkpoint separate with replay-safe key. Explain unavoidable failure window.

**Write in your notebook:** input; expected output; actual output; why it happened; validation evidence; what you would do if it failed.

## Lab cell 6

Poison row: isolate only if rule says row-level reject is safe. If every row fails due schema mismatch, fail the entire batch and alert.

**Write in your notebook:** input; expected output; actual output; why it happened; validation evidence; what you would do if it failed.

## Lab cell 7

Idempotency test: run same input twice; output row count and target values remain same. Test same key with changed source version too.

**Write in your notebook:** input; expected output; actual output; why it happened; validation evidence; what you would do if it failed.

## Lab cell 8

Recovery runbook: identify failed run and safe cursor; inspect partial destination; repair cause; replay bounded unit; reconcile; record recovered status.

**Write in your notebook:** input; expected output; actual output; why it happened; validation evidence; what you would do if it failed.

# 15. Lab completion evidence

| Evidence | Student submission |
|---|---|
| Input contract | Fields, types, keys, source behavior |
| Happy-path run | Input and output counts |
| Invalid case | Error/reject reason and handling |
| Replay | Proof no duplicate side effect |
| Boundary case | Proof no gap or unintended overlap |
| Recovery | Last safe point and restart procedure |
| Reconciliation | Counts, keys, totals or quality checks |
| Project defence | Explain decision, risk, control and proof |



---

# Guided execution lab — Reliable Pipeline Simulator (runnable fixture)

This is a deterministic simulation of a retry-safe load. It demonstrates the key design: stable record identity, idempotent writes, bounded retry, and checkpoint only after success.

## Cell 1 — Source batch

```python
source_batch = [
    {"event_id": "E1", "order_id": 501, "amount": 100},
    {"event_id": "E2", "order_id": 502, "amount": 200},
    {"event_id": "E3", "order_id": 503, "amount": 300}
]
target_by_event = {}
checkpoint = 0
```

`event_id` identifies an event uniquely. `target_by_event` acts like a destination table with a unique key.

## Cell 2 — Idempotent write

```python
for event in source_batch:
    target_by_event[event["event_id"]] = event
print("Rows in target:", len(target_by_event))
```

Expected: 3. A dictionary assignment by stable key replaces the same event rather than adding a duplicate.

## Cell 3 — Replay the same batch

```python
for event in source_batch:
    target_by_event[event["event_id"]] = event
print("Rows after replay:", len(target_by_event))
```

Expected: still 3. This illustrates idempotent effect for this key/value operation. It does not prove every external side effect is idempotent.

## Cell 4 — Simulate failure before checkpoint

```python
checkpoint = 0
print("Target persisted rows:", len(target_by_event))
print("Checkpoint:", checkpoint)
```

The target has data but checkpoint has not advanced. On restart, the batch is replayed. Because writes use stable event IDs, replay does not duplicate the target records.

## Cell 5 — Verify then checkpoint

```python
expected_ids = {event["event_id"] for event in source_batch}
actual_ids = set(target_by_event.keys())
assert expected_ids.issubset(actual_ids)
checkpoint = 3
print("Verified:", True)
print("Checkpoint:", checkpoint)
```

Expected: Verified True; Checkpoint 3. In production, checkpoint and target commit should be coordinated transactionally where possible; otherwise use an outbox/manifest/recovery protocol.

## Cell 6 — Bounded retry policy

A retry should be limited and only used for transient failures (temporary network error, throttling, service unavailable). Do not retry permanent errors such as invalid schema forever. Use exponential backoff with jitter, a maximum attempt count, timeout and dead-letter/quarantine path.

## Failure drill

1. Fail before any target write: replay the full batch.
2. Fail after E1 and E2: replay all; stable keys prevent duplicates.
3. Change E2 amount and same event ID: decide whether events are immutable or versioned. Do not silently overwrite immutable event history.
4. Make a malformed record: route to reject/quarantine, don't retry a permanent validation failure.
5. Confirm checkpoint never indicates progress beyond durable target data.
