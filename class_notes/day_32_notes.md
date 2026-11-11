# Day 32: API-Based Data Ingestion

**Phase:** 4 — Data Processing and Pipelines  
**Syllabus topic:** API-Based Data Ingestion  
**Mini-project:** API Ingestion Pipeline  
**Learning time:** 2–3 hours  
**Level:** Complete beginner to junior Data Engineer  

---

# 1. What are we learning today?

A marketplace API returns limited pages, may fail temporarily and requires credentials. Build a polite, restartable extractor.

## 1.1 Learning outcomes

By the end of this lesson, you should be able to:

1. Read API documentation and permitted usage.
2. Keep credentials in environment/secret storage, never source code.
3. Check status before parsing JSON; validate keys/types.
4. Follow page tokens until the documented end; detect repeated cursors.
5. Handle 429/Retry-After and transient 5xx with bounded backoff.
6. Persist each page safely; checkpoint only after durable write.
7. Reconcile page counts, IDs and duplicate records.

## 1.2 Pipeline context

```text
Business source -> API-Based Data Ingestion -> Validate/process -> Staging/destination -> Reconcile -> Checkpoint + monitoring
```

A pipeline is not merely a script that reads data. It is repeatable work with defined input, rules, output, progress, error path and evidence of correctness.

# 2. Business scenario and vocabulary

A marketplace API returns limited pages, may fail temporarily and requires credentials. Build a polite, restartable extractor.

| Term | Beginner meaning | Why it matters |
|---|---|---|
| **API** |  contract between systems | Used to design and explain reliable pipelines. |
| **endpoint** |  resource URL | Used to design and explain reliable pipelines. |
| **GET** |  retrieve | Used to design and explain reliable pipelines. |
| **status code** |  request result | Used to design and explain reliable pipelines. |
| **pagination** |  split results | Used to design and explain reliable pipelines. |
| **rate limit** |  request quota | Used to design and explain reliable pipelines. |
| **retry** |  controlled repeat | Used to design and explain reliable pipelines. |
| **cursor/checkpoint** |  saved progress. | Used to design and explain reliable pipelines. |

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

Demonstrate rest apis with one small operation before scaling it into a pipeline.

## Code

```python
import requests
response = requests.get("https://api.example.com/orders", timeout=20)
print(response.status_code)
print(response.json())
```

## Expected output

```text
200
{'orders': [{'order_id': 501, 'amount': 1200}], 'next_page': 2}
```

> Sample output is for the stated sample values. API/database/file results may vary. Do not claim variable outputs are guaranteed or runtime-tested unless you executed them.

## Line-by-line explanation

GET sends a request; timeout prevents indefinite waiting; status_code reports HTTP outcome; json() parses the body but does not validate business data.

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

## Mistake 1: Parse JSON before status check

**Why dangerous:** Error responses may not be JSON or hide cause.

**Correct approach:** Check response status, then parse.

## Mistake 2: Infinite pagination

**Why dangerous:** Cursor does not advance or repeats.

**Correct approach:** Track cursor and stop using provider contract.

## Mistake 3: Retry all 4xx

**Why dangerous:** Invalid credentials/parameters are permanent.

**Correct approach:** Retry only transient failures and 429.

## Mistake 4: No timeout

**Why dangerous:** A call can hang forever.

**Correct approach:** Set connection/read timeout.

## Mistake 5: Save checkpoint before output

**Why dangerous:** Crash can skip records.

**Correct approach:** Persist first; checkpoint after durable success.


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

# 7. Mini-project — API Ingestion Pipeline

## Objective

Build a beginner-readable pipeline demonstrating api-based data ingestion with a happy path, invalid/failure path and verification step.

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

Explain API to a beginner; give a business example; name a failure; explain detection and recovery.

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

# 10. Day 32 revision sheet

**Flow:** Define contract -> Read bounded input -> Validate -> Process -> Persist -> Verify -> Checkpoint -> Observe

Never skip: unique record identity; what success guarantees; replay behavior; invalid versus transient error; evidence of destination correctness.

## One-minute project defence

“My API Ingestion Pipeline receives data from a defined source. It validates the contract and identity, processes controlled units, handles invalid/transient cases explicitly, persists safely, records progress after durable success and reconciles counts/values. I can rerun it safely and diagnose failures from run metadata. Before production I would add automated tests, secret management, monitoring, access controls and scale testing.”

## Self-assessment

- [ ] I can define terms without memorizing.
- [ ] I can explain code and output.
- [ ] I can trace a record end-to-end.
- [ ] I can reproduce/debug a failure.
- [ ] I can explain checkpoint ordering and replay.
- [ ] I can prove correctness with checks.
- [ ] I can defend the project.

**Complete the day when you can explain, run, interpret, test failure and prove the result—not merely read the notes.**


# 11. Mentor-led deep dive — API

This section expands the earlier example into decisions a junior Data Engineer should make. Read each scenario slowly: identify the input, risk, control and proof.

## 11.1 Request URL anatomy

A URL can contain a base host, path and query parameters. For example, /orders?limit=100&status=PAID asks the provider for a filtered/limited set. The provider—not Python—defines what each parameter means. Never guess undocumented parameters.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.2 HTTP response anatomy

A response has status, headers and body. Status tells request outcome; headers can include content type, pagination/rate-limit information; body contains the payload. Inspect these separately while debugging.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.3 Safe request walkthrough

First print status and content type for a public test endpoint. Then parse JSON. Next validate the expected top-level key. Finally validate one record. This order separates network failure, parsing failure and data-contract failure.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.4 Authentication teaching

A token is a credential, not ordinary configuration. Read it from an environment variable or secret manager; do not commit it, paste it in notebook output or include it in exception messages. Rotate it if exposed.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.5 Pagination example

Suppose page 1 returns IDs 1–3 and next_page=2; page 2 returns 4–6 and next_page=null. Your loop should fetch page 2 exactly once, append only validated results and stop when the provider's terminal marker is reached.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.6 Cursor safety

A cursor is opaque. Treat it as a string supplied by the API; do not calculate cursor+1 unless documentation explicitly says it is numeric. Store the exact token safely.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.7 Rate-limit response

When 429 occurs, stop sending immediate requests. Read Retry-After if available, wait, then resume. A tight retry loop makes the provider's problem worse.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.8 Retry classification

Timeout/temporary 503 can be retried. 400 invalid input, 401 credentials and 403 permission normally need correction. 404 may mean endpoint/resource is absent. Make the classification explicit.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.9 Incremental API boundary

If provider offers updated_since, understand whether it is inclusive, timezone-aware and stable. Use overlap/lookback when events can arrive late; deduplicate by stable event/order ID.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.10 API output validation

Check response is a dictionary, expected collection key exists, collection is a list, each item has an ID and required fields, and values have expected types. JSON can be valid but semantically wrong.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.11 Backfill design

For 90-day history, split into date windows or cursor batches. Keep each batch idempotent, record window status, limit provider load, and resume failed windows without starting all over.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.12 Streaming introduction

Streaming differs from paginated request/response. A connection can disconnect; use provider offsets/event IDs, reconnect and replay. Persist events before acknowledging progress.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.13 Project extension

Add a mock response fixture for success, empty page, 429, timeout, malformed JSON and missing field. Students can test failure behavior without repeatedly calling a real API.

**Student checkpoint:** Explain this in your own words, then name one test that would prove it.

## 11.14 Interview evidence

Show request parameters, status validation, page count, unique IDs, retry count, checkpoint and final reconciliation. A screenshot of one successful response is weak evidence.

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

