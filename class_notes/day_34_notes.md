# Day 34: Data Transformation and Business Rules

**Phase:** 4 — Data Processing and Pipelines  
**Syllabus topic:** Data Transformation and Business Rules  
**Mini-project:** Transformation Pipeline  
**Learning time:** 2–3 hours  
**Level:** Complete beginner to junior Data Engineer  

---

# 1. What are we learning today?

Raw orders contain inconsistent statuses and types. Standardize them without silently changing meaning.

## 1.1 Learning outcomes

By the end of this lesson, you should be able to:

1. Profile raw values first.
2. Document each business rule and owner.
3. Normalize only where semantics permit.
4. Convert types explicitly; capture conversion failures.
5. Map unknown codes to exception path.
6. Calculate derived fields with precision policy.
7. Validate ranges, required values and cross-column rules.
8. Preserve raw data and transformation version.
9. Reconcile counts and totals.

## 1.2 Pipeline context

```text
Business source -> Data Transformation and Business Rules -> Validate/process -> Staging/destination -> Reconcile -> Checkpoint + monitoring
```

A pipeline is not merely a script that reads data. It is repeatable work with defined input, rules, output, progress, error path and evidence of correctness.

# 2. Business scenario and vocabulary

Raw orders contain inconsistent statuses and types. Standardize them without silently changing meaning.

| Term | Beginner meaning | Why it matters |
|---|---|---|
| **Transformation** |  explicit data change | Used to design and explain reliable pipelines. |
| **standardization** |  consistent representation | Used to design and explain reliable pipelines. |
| **business rule** |  business logic | Used to design and explain reliable pipelines. |
| **mapping** |  source-to-target code | Used to design and explain reliable pipelines. |
| **derived column** |  calculated value | Used to design and explain reliable pipelines. |
| **lineage** |  source-to-output history | Used to design and explain reliable pipelines. |
| **validation** |  check expectations | Used to design and explain reliable pipelines. |
| **quarantine** |  isolate bad records. | Used to design and explain reliable pipelines. |

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

Demonstrate cleaning with one small operation before scaling it into a pipeline.

## Code

```python
import pandas as pd
data = pd.DataFrame({"status": ["paid", "Paid", "cancelled"]})
data["status"] = data["status"].str.strip().str.upper()
print(data)
```

## Expected output

```text
      status
0      PAID
1      PAID
2  CANCELLED
```

> Sample output is for the stated sample values. API/database/file results may vary. Do not claim variable outputs are guaranteed or runtime-tested unless you executed them.

## Line-by-line explanation

strip removes surrounding spaces; upper normalizes casing. Unknown status codes must be reported, not blindly assigned meaning.

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

## Mistake 1: NULL replaced with zero

**Why dangerous:** Unknown differs from known zero.

**Correct approach:** Define field-specific null/default/reject policy.

## Mistake 2: Drop unknown statuses

**Why dangerous:** Hides source changes.

**Correct approach:** Retain source value and alert/quarantine.

## Mistake 3: Round early

**Why dangerous:** Financial precision loss.

**Correct approach:** Use Decimal and round at approved stage.

## Mistake 4: Mutate raw input

**Why dangerous:** Replay/audit becomes difficult.

**Correct approach:** Preserve raw and produce transformed layer.

## Mistake 5: No rule version

**Why dangerous:** Outputs cannot be reproduced.

**Correct approach:** Record code/rule version per run.


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

# 7. Mini-project — Transformation Pipeline

## Objective

Build a beginner-readable pipeline demonstrating data transformation and business rules with a happy path, invalid/failure path and verification step.

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

Explain Transformation to a beginner; give a business example; name a failure; explain detection and recovery.

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

# 10. Day 34 revision sheet

**Flow:** Define contract -> Read bounded input -> Validate -> Process -> Persist -> Verify -> Checkpoint -> Observe

Never skip: unique record identity; what success guarantees; replay behavior; invalid versus transient error; evidence of destination correctness.

## One-minute project defence

“My Transformation Pipeline receives data from a defined source. It validates the contract and identity, processes controlled units, handles invalid/transient cases explicitly, persists safely, records progress after durable success and reconciles counts/values. I can rerun it safely and diagnose failures from run metadata. Before production I would add automated tests, secret management, monitoring, access controls and scale testing.”

## Self-assessment

- [ ] I can define terms without memorizing.
- [ ] I can explain code and output.
- [ ] I can trace a record end-to-end.
- [ ] I can reproduce/debug a failure.
- [ ] I can explain checkpoint ordering and replay.
- [ ] I can prove correctness with checks.
- [ ] I can defend the project.

**Complete the day when you can explain, run, interpret, test failure and prove the result—not merely read the notes.**


# 11. Mentor-led deep dive — Transformation

Read each scenario slowly. Identify input, risk, control and proof.

## 11.1 Raw/curated distinction

Raw versus curated: preserve source representation and provenance in raw layer; apply documented rules in curated layer so replay remains possible.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.2 Walkthrough

Type conversion: parse CSV amount '1200.50' as Decimal, handle blank/invalid explicitly, validate range and write decimal target; don't let implicit conversion hide bad values.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.3 Normalization policy

Normalize whitespace/casing only where semantics are case-insensitive; preserve names/address source meaning.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.4 Mapping design

Mapping table: source_status,target_status,valid_from,valid_to,owner; unknown code becomes observable exception.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.5 Worked business example

Derived total: quantity 3 × unit_price 100.00 = gross 300.00; discount 10.00 => net 290.00. Explain units, precision and null behavior.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.6 Filter semantics

Filtering removes downstream records. Record why/count; SQL comparisons with NULL can evaluate unknown and exclude rows.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.7 Aggregation grain

Aggregate at intended grain (order/day/product); validate keys and dedup first to prevent inflated sums.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.8 Transformation order

Order: parse -> validate -> standardize -> map -> derive -> aggregate -> output; document exceptions.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.9 Reject evidence

Reject schema: source ID, original safe reference, run ID, rule, error and timestamp; avoid unnecessary sensitive values.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.10 Test design

Test normal, boundary, null, invalid, unknown mapping and duplicate; assert exact output/count invariant.

**Student checkpoint:** Explain this in your own words and name a test that proves it.

## 11.11 Reconciliation

Reconcile raw = accepted + rejected; compare totals with documented exclusions; monitor unknown categories/null-rate shifts.

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

