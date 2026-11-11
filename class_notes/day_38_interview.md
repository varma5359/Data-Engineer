# Day 38 Interview Guide — Pipeline Reliability and Failure Handling

**Project:** TechStore Reliable Pipeline Simulator  
**Audience:** Fresher / Junior Data Engineer  
**Interview method:** Explain the failure, identify the risk, describe the control, then state how you would prove recovery.

---

## Part 1 — Core concepts with complete answers

### 1. What is pipeline reliability?

**Model answer:** Pipeline reliability means the pipeline consistently produces correct and complete results within agreed operational limits, and can recover safely when something fails. It is not simply that the script runs without an exception. I evaluate correctness, completeness, repeatability, recoverability, timeliness and operational visibility.

### 2. What is a transient failure?

**Model answer:** A transient failure is temporary and may succeed if the operation is retried later. Examples include a brief network interruption, database connection reset or HTTP 503 from an overloaded service. A bounded retry with backoff may help, provided the operation is safe to repeat.

### 3. What is a permanent failure?

**Model answer:** A permanent failure requires a correction rather than another identical attempt. Examples include invalid SQL, missing required columns, expired credentials, insufficient permissions or malformed input. Retrying these errors repeatedly wastes resources and can make an incident worse.

### 4. What is a retry policy?

**Model answer:** A retry policy defines which failures may be retried, the maximum number of attempts, timeout per attempt, delay between attempts, backoff strategy, jitter and what happens after the final failure. A retry policy should be explicit and observable. It should not retry every exception indiscriminately.

### 5. What is exponential backoff?

**Model answer:** Exponential backoff increases the wait between successive retries, for example 1, 2, 4 and 8 seconds, often with a maximum cap. It reduces repeated pressure on a service that may be recovering. The exact schedule depends on the dependency and its retry guidance.

### 6. Why add jitter to backoff?

**Model answer:** Jitter adds a random variation to retry delays. Without it, many clients may retry at exactly the same time after an outage and create a retry storm. Jitter spreads requests over time and reduces synchronized load.

### 7. Why should retries be bounded?

**Model answer:** Infinite retries can hold workers forever, consume connections, overload dependencies and hide permanent errors. A bounded policy stops after a defined number of attempts, records the failure and alerts or escalates. The pipeline should fail visibly rather than report false success.

### 8. What is a timeout?

**Model answer:** A timeout limits how long an operation is allowed to wait. A connection timeout limits time to establish a connection; a read timeout limits waiting for response data; a task timeout limits the overall operation. Timeouts prevent a pipeline from hanging indefinitely and holding resources.

### 9. What is idempotent processing?

**Model answer:** Processing is idempotent when repeating the same input produces the same final state as processing it once. A target upsert using a stable unique key can be idempotent. Blindly appending the same records is not. Idempotency is essential for safe retries and replay after uncertain failures.

### 10. What is an atomic write?

**Model answer:** An atomic write ensures consumers see either the complete committed result or no result, rather than a partially published result. In a database this may be achieved with a transaction. For files, a common pattern is to write to a temporary path, validate, then publish using an atomic rename where the storage system supports it. Object stores may require manifest or pointer-based publication rather than assuming rename is atomic.

### 11. What is a partial failure?

**Model answer:** A partial failure occurs when some stages or batches succeed and later work fails. For example, six of ten batches may be written before the seventh fails. The run must not be marked complete. I need a durable record of completed work, staging or transaction boundaries, reconciliation and a safe resume or replay strategy.

### 12. What is a checkpoint?

**Model answer:** A checkpoint records the last safely completed point in processing. It allows a restarted pipeline to resume without starting from the beginning. The checkpoint must correspond to work that has been committed and verified. If checkpoint and target writes are separate operations, idempotency is needed to handle uncertain outcomes.

### 13. What is a dead-letter queue?

**Model answer:** A dead-letter queue or quarantine destination stores messages or records that cannot be processed successfully after the defined policy. It should include the original payload or a safe reference, failure reason, attempt count, source identity and timestamps. It supports investigation and controlled replay. It should not become a place where bad records are forgotten.

### 14. What is a poison message?

**Model answer:** A poison message is a record or event that repeatedly causes processing to fail, often because it is malformed or triggers a software defect. If it blocks an entire batch or partition, the pipeline may never progress. A controlled dead-letter or quarantine strategy isolates it while preserving evidence.

### 15. What is a retry storm?

**Model answer:** A retry storm happens when many clients repeatedly retry a failing dependency, adding load while it is already unhealthy. Bounded retries, exponential backoff, jitter, rate limits and circuit breakers reduce this risk.

### 16. What is a circuit breaker?

**Model answer:** A circuit breaker temporarily stops calls to a dependency after a failure threshold is reached. It prevents repeated calls to a service that is clearly unhealthy. After a cool-down period, it may allow a limited test request. It complements retries; it does not replace correct error classification.

### 17. Why should we avoid claiming exactly-once processing casually?

**Model answer:** Failures can occur between writing data and acknowledging or saving progress. A system may perform a side effect and crash before recording that it succeeded, causing replay. Exactly-once behavior requires guarantees across the relevant source, processing and destination boundaries. In many pipelines, at-least-once delivery plus idempotent writes and deduplication is the practical design. I describe the actual guarantee rather than using an unsupported label.

### 18. How do transactions improve reliability?

**Model answer:** A database transaction groups related writes so they commit together or roll back together. It prevents consumers from seeing a partial set of changes within that transaction boundary. It does not automatically make external API calls or file writes part of the same transaction, so cross-system workflows still need idempotency and recovery design.

### 19. How do you decide whether to continue after a bad record?

**Model answer:** I follow the data contract and business criticality. If the record is independently invalid and can be safely isolated, I may quarantine it and continue while recording counts. If the record affects financial totals, referential integrity or a critical batch-level invariant, the entire batch may need to fail. The decision must be explicit and measurable.

### 20. How do you test recovery?

**Model answer:** I inject controlled failures at important boundaries: source timeout, malformed row, destination unavailable, crash after some batches, crash after target commit but before checkpoint, duplicate replay and checkpoint-store failure. After each test I verify target state, checkpoint state, accepted/rejected counts and whether rerun produces duplicates or data loss.

---

## Part 2 — Project-based questions

### 21. Explain the TechStore Reliable Pipeline Simulator.

**Model answer:** The project begins with a small batch of events and an empty target. It writes records using stable event IDs so the same batch can be replayed without increasing the target count. It simulates a transient read failure and bounded retry, distinguishes permanent data errors, demonstrates checkpoint ordering, and quarantines a bad record. The project is intentionally small so each failure and recovery result can be inspected.

### 22. How did you demonstrate idempotency?

**Model answer:** I wrote the same batch twice using event_id as the target key. After the first write the target contained three events. After replay, it still contained three events with the same final values. This demonstrates idempotent final state for this deterministic example. It does not by itself prove that every external side effect is exactly once.

### 23. What does your simulator not prove?

**Model answer:** It is a teaching simulation, not a distributed production system. A Python dictionary does not provide database durability, concurrent transaction isolation or cross-system atomicity. The exercise teaches the logic of stable keys, replay, retry and checkpoint order. A production implementation would use durable storage, transactions, concurrency controls, monitoring and integration tests.

### 24. How would you improve the simulator for production?

**Model answer:** I would persist run and checkpoint state, use a database unique constraint, wrap related writes in transactions, add structured logs and metrics, classify retryable exceptions, use backoff with jitter, isolate invalid events, and test crash recovery at each commit boundary. I would also document the delivery guarantee and add alerts for repeated failures.

---

## Part 3 — Scenario questions with strong answers

### 25. The API returns HTTP 503. What do you do?

**Strong answer:** I treat 503 as potentially transient, inspect any Retry-After guidance, apply a bounded retry with backoff and jitter, and record attempts. If the limit is reached, I fail the run visibly and preserve the last safe checkpoint. I do not advance progress for data that was not successfully loaded.

### 26. The API returns HTTP 400 because a required parameter is missing.

**Strong answer:** This is generally a request/configuration defect, not a transient service problem. I do not repeat the same request. I log a safe error context, stop the stage, correct the parameter or code and rerun from a safe boundary.

### 27. The process crashes after writing half the output.

**Strong answer:** I prevent partial output from being published. I write into a run-specific staging area or use a transaction, track completed batches, validate the final result and publish only after reconciliation. On restart I resume or replay using idempotent writes. I inspect whether any partial output was visible to consumers.

### 28. The target transaction committed, but the process crashed before saving the checkpoint.

**Strong answer:** I expect the source interval to be read again. The target write must be idempotent, using a stable key or deduplication mechanism. I replay, verify the target state and then save the checkpoint. I include this exact failure boundary in tests.

### 29. A single malformed record repeatedly fails an entire daily batch.

**Strong answer:** I determine whether the record can be isolated safely. If it violates a row-level rule, I send it to quarantine with the payload or secure reference and reason, then continue if the contract permits. If it violates a batch-level invariant, I fail the batch. I track the poison record so it does not disappear silently.

### 30. The database is unavailable for 30 minutes. Should the job retry every second?

**Strong answer:** No. Frequent retries can overload the database during recovery and consume worker capacity. I use bounded backoff with jitter, respect service guidance, set a maximum elapsed time and alert when the retry budget is exhausted. I preserve the last safe checkpoint and resume when the dependency recovers.

### 31. Two workers process the same event simultaneously. How do you prevent duplicates?

**Strong answer:** I use a database-enforced unique key or idempotency key, not only an in-memory check. I define transaction behavior for conflicts and ensure concurrent upserts are safe. I test simultaneous processing and inspect the final target state. Application-level checks alone can race.

### 32. A retry succeeds but creates duplicate emails or payments.

**Strong answer:** The operation has an external side effect that is not idempotent. I need an idempotency token or durable request ledger accepted by the external service, or a workflow that checks whether the side effect already occurred before repeating it. A database upsert cannot automatically make an external payment exactly once.

---

## Part 4 — Practical revision checklist

- Classify transient and permanent failures.
- Explain bounded retry, exponential backoff and jitter.
- Explain idempotency and safe replay.
- Explain partial writes and atomic publication.
- Explain checkpoint ordering.
- Describe a dead-letter/quarantine record.
- Explain why exactly-once claims need qualification.
- Describe at least four failure-injection tests.
- State what the teaching simulator does not prove.

## Part 5 — Self-assessment

| Skill | Score (0–2) |
|---|---:|
| Failure classification | /2 |
| Timeouts and retries | /2 |
| Backoff and jitter | /2 |
| Idempotency | /2 |
| Atomic writes | /2 |
| Checkpoint recovery | /2 |
| Quarantine / DLQ | /2 |
| Failure testing | /2 |

**13–16:** ready for junior-level discussion. **9–12:** repeat the failure drills. **0–8:** rebuild the simulator and explain each state transition.
