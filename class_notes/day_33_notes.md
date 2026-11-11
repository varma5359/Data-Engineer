# Day 33: Database Ingestion

**Phase:** 4 — Data Processing and Pipelines  
**Syllabus topic:** Database Ingestion  
**Mini-project:** Database Extractor  
**Learning time:** 2–3 hours  
**Level:** Complete beginner to junior Data Engineer  

---

# 1. What are we learning today?

Copy operational data without overwhelming source DB; extract with stable progress and reconcile source/target.

## 1.1 Learning outcomes

By the end of this lesson, you should be able to:

1. Identify table, primary key, change column, volume and source limits.
2. Use least-privilege read-only account.
3. Select explicit columns and deterministic ORDER BY.
4. Use driver placeholders, never concatenate values.
5. Fetch in bounded batches for large tables.
6. Choose full/incremental strategy and safe watermark.
7. Write idempotently and checkpoint after target commit.
8. Reconcile counts, keys, totals and boundaries.

## 1.2 Pipeline context

```text
Business source -> Database Ingestion -> Validate/process -> Staging/destination -> Reconcile -> Checkpoint + monitoring
```

A pipeline is not merely a script that reads data. It is repeatable work with defined input, rules, output, progress, error path and evidence of correctness.

# 2. Business scenario and vocabulary

Copy operational data without overwhelming source DB; extract with stable progress and reconcile source/target.

| Term | Beginner meaning | Why it matters |
|---|---|---|
| **Source DB** |  system of record | Used to design and explain reliable pipelines. |
| **connection** |  session | Used to design and explain reliable pipelines. |
| **cursor** |  SQL execution/fetch object | Used to design and explain reliable pipelines. |
| **watermark** |  extraction boundary | Used to design and explain reliable pipelines. |
| **snapshot** |  consistent view | Used to design and explain reliable pipelines. |
| **read replica** |  copy for reads | Used to design and explain reliable pipelines. |
| **parameterized query** |  values separate from SQL. | Used to design and explain reliable pipelines. |

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

Demonstrate database extraction with one small operation before scaling it into a pipeline.

## Code

```python
import psycopg
with psycopg.connect("dbname=shop user=student password=student") as conn:
    with conn.cursor() as cur:
        cur.execute("SELECT order_id, amount FROM orders ORDER BY order_id")
        rows = cur.fetchall()
        print(rows)
```

## Expected output

```text
[(101, Decimal('1200.00')), (102, Decimal('850.00')), (103, Decimal('499.00'))]
```

> Sample output is for the stated sample values. API/database/file results may vary. Do not claim variable outputs are guaranteed or runtime-tested unless you executed them.

## Line-by-line explanation

Connection/cursor context managers manage resources; execute runs SQL; fetchall materializes all rows and is suitable only for small examples, not huge extracts.

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

## Mistake 1: fetchall on millions

**Why dangerous:** Memory grows with result.

**Correct approach:** fetchmany/server-side cursor/key batches.

## Mistake 2: ID-only incremental

**Why dangerous:** Misses updates to old IDs.

**Correct approach:** Use updated_at, CDC or reconciliation.

## Mistake 3: SQL concatenation

**Why dangerous:** Injection and quoting risk.

**Correct approach:** Parameterized SQL.

## Mistake 4: Long source transaction

**Why dangerous:** Can affect locks/snapshots.

**Correct approach:** Bound extraction and coordinate with DBA.

## Mistake 5: No delete capture

**Why dangerous:** Deleted source rows remain target.

**Correct approach:** CDC/tombstone or periodic reconciliation.


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

# 7. Mini-project — Database Extractor

## Objective

Build a beginner-readable pipeline demonstrating database ingestion with a happy path, invalid/failure path and verification step.

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

Explain Source DB to a beginner; give a business example; name a failure; explain detection and recovery.

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

# 10. Day 33 revision sheet

**Flow:** Define contract -> Read bounded input -> Validate -> Process -> Persist -> Verify -> Checkpoint -> Observe

Never skip: unique record identity; what success guarantees; replay behavior; invalid versus transient error; evidence of destination correctness.

## One-minute project defence

“My Database Extractor receives data from a defined source. It validates the contract and identity, processes controlled units, handles invalid/transient cases explicitly, persists safely, records progress after durable success and reconciles counts/values. I can rerun it safely and diagnose failures from run metadata. Before production I would add automated tests, secret management, monitoring, access controls and scale testing.”

## Self-assessment

- [ ] I can define terms without memorizing.
- [ ] I can explain code and output.
- [ ] I can trace a record end-to-end.
- [ ] I can reproduce/debug a failure.
- [ ] I can explain checkpoint ordering and replay.
- [ ] I can prove correctness with checks.
- [ ] I can defend the project.

**Complete the day when you can explain, run, interpret, test failure and prove the result—not merely read the notes.**


# 11. Mentor-led deep dive — DB

This section expands the earlier example into decisions a junior Data Engineer should make. Read each scenario slowly: identify the input, risk, control and proof.

## 11.1 Why source extraction needs care

Operational systems serve customer transactions. Large scans, long snapshots or expensive joins can slow checkout/booking. Discuss extraction windows, read replicas, source-side filters and DBA approval.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.2 Connection setup

Install the matching driver, use connection settings from environment/config, open a connection, create cursor, execute a query, fetch rows and close resources. Explain each object before adding pipeline logic.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.3 Parameterized SQL walkthrough

SQL text contains placeholders; values are passed as a separate tuple/list. Placeholder syntax depends on driver. Parameters are values—not table/column names—so dynamic identifiers require safe allow-list handling.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.4 Batch fetch

Fetch 500 rows, process/write them, then fetch next batch. Do not accumulate every batch into one giant list. Track count and last key; test interruption between batches.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.5 Keyset pagination

For append-only ID: WHERE id > last_id AND id <= upper_id ORDER BY id LIMIT batch_size. Capture upper_id at start to bound the run. Explain why updates/deletes still need CDC or another method.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.6 Timestamp compound cursor

Use WHERE updated_at > last_time OR (updated_at = last_time AND id > last_id), ordered by updated_at,id. Explain how this resolves timestamp ties and why updates that move timestamps matter.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.7 Source snapshot

If records change while extraction runs, a query may observe different moments. Snapshot isolation/consistent snapshot/CDC strategies depend on database and operational constraints; don't claim a plain SELECT is always consistent.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.8 Read replica lag

Replica can be behind primary. A watermark taken from primary then queried on lagging replica can skip rows unless overlap/replication position is considered.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.9 Data type mapping

SQL DECIMAL may become Decimal in Python; timestamp may become datetime; NULL becomes None. Explain these conversions and avoid float for money.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.10 Large-table extraction

Select only required columns, filter in SQL, use bounded batches and index-friendly predicates. Monitor duration, rows, source load and query plan.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.11 Connection failure

If connection drops, identify last durable target batch. Reconnect and replay from checkpoint; target writes must be idempotent. Do not assume in-memory last row is durable progress.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.12 Delete capture

Hard-deleted rows vanish from SELECT. Use WAL/binlog CDC, soft-delete flag, tombstone table or periodic reconciliation.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.13 Project test matrix

Test empty table, one row, duplicate key, NULL timestamp, timestamp tie, update old ID, deleted source row, timeout and restart.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

# 12. Guided lab checkpoints

## Lab checkpoint 1

- **Predict:** Before running anything, write what you expect to happen.
- **Execute:** Run only this stage with a small input.
- **Observe:** Record the actual output, row count and error (if any).
- **Explain:** Why did the output occur? Which input or rule produced it?
- **Verify:** Compare it with the expected result and contract.
- **Recover:** If it failed, reproduce with the smallest input and correct the cause.

## Lab checkpoint 2

- **Predict:** Before running anything, write what you expect to happen.
- **Execute:** Run only this stage with a small input.
- **Observe:** Record the actual output, row count and error (if any).
- **Explain:** Why did the output occur? Which input or rule produced it?
- **Verify:** Compare it with the expected result and contract.
- **Recover:** If it failed, reproduce with the smallest input and correct the cause.

## Lab checkpoint 3

- **Predict:** Before running anything, write what you expect to happen.
- **Execute:** Run only this stage with a small input.
- **Observe:** Record the actual output, row count and error (if any).
- **Explain:** Why did the output occur? Which input or rule produced it?
- **Verify:** Compare it with the expected result and contract.
- **Recover:** If it failed, reproduce with the smallest input and correct the cause.

## Lab checkpoint 4

- **Predict:** Before running anything, write what you expect to happen.
- **Execute:** Run only this stage with a small input.
- **Observe:** Record the actual output, row count and error (if any).
- **Explain:** Why did the output occur? Which input or rule produced it?
- **Verify:** Compare it with the expected result and contract.
- **Recover:** If it failed, reproduce with the smallest input and correct the cause.

## Lab checkpoint 5

- **Predict:** Before running anything, write what you expect to happen.
- **Execute:** Run only this stage with a small input.
- **Observe:** Record the actual output, row count and error (if any).
- **Explain:** Why did the output occur? Which input or rule produced it?
- **Verify:** Compare it with the expected result and contract.
- **Recover:** If it failed, reproduce with the smallest input and correct the cause.

## Lab checkpoint 6

- **Predict:** Before running anything, write what you expect to happen.
- **Execute:** Run only this stage with a small input.
- **Observe:** Record the actual output, row count and error (if any).
- **Explain:** Why did the output occur? Which input or rule produced it?
- **Verify:** Compare it with the expected result and contract.
- **Recover:** If it failed, reproduce with the smallest input and correct the cause.

## Lab checkpoint 7

- **Predict:** Before running anything, write what you expect to happen.
- **Execute:** Run only this stage with a small input.
- **Observe:** Record the actual output, row count and error (if any).
- **Explain:** Why did the output occur? Which input or rule produced it?
- **Verify:** Compare it with the expected result and contract.
- **Recover:** If it failed, reproduce with the smallest input and correct the cause.

## Lab checkpoint 8

- **Predict:** Before running anything, write what you expect to happen.
- **Execute:** Run only this stage with a small input.
- **Observe:** Record the actual output, row count and error (if any).
- **Explain:** Why did the output occur? Which input or rule produced it?
- **Verify:** Compare it with the expected result and contract.
- **Recover:** If it failed, reproduce with the smallest input and correct the cause.


# 13. Instructor review rubric

| Skill | Beginner evidence |
|---|---|
| Concept | Explains what/why in plain language |
| Code | Can read and explain each line, not copy blindly |
| Output | Can derive output from input and rule |
| Quality | Identifies invalid, duplicate and boundary cases |
| Reliability | Explains retry, replay and checkpoint |
| Verification | Uses counts/keys/totals/quality checks |
| Interview | Gives decision, reason, failure mode and proof |

