# Day 38 Interview Preparation: Pipeline Reliability and Failure Handling

**Mini-project:** Reliable Pipeline Simulator  
**Target:** Fresher / Junior Data Engineer  
**Method:** Answer aloud first; compare with the model answer; then connect it to your project.

---

# 1. Core concepts — complete model answers

## 1. What is pipeline reliability?

**Model answer:** Consistently produces correct, complete, recoverable results within expected operational limits.

**Follow-up:** Give a concrete example from Reliable Pipeline Simulator; state what evidence proves your answer.

## 2. Transient versus permanent?

**Model answer:** Transient may recover (timeout/503); permanent needs correction (schema/credentials/bad request).

**Follow-up:** Give a concrete example from Reliable Pipeline Simulator; state what evidence proves your answer.

## 3. What is retry policy?

**Model answer:** Eligible errors, attempts, delays, jitter, timeout and terminal behavior.

**Follow-up:** Give a concrete example from Reliable Pipeline Simulator; state what evidence proves your answer.

## 4. Why bounded retries?

**Model answer:** Infinite retries consume resources and hide outages; bound and alert.

**Follow-up:** Give a concrete example from Reliable Pipeline Simulator; state what evidence proves your answer.

## 5. What is exponential backoff?

**Model answer:** Increasing delay, often with jitter, to reduce pressure.

**Follow-up:** Give a concrete example from Reliable Pipeline Simulator; state what evidence proves your answer.

## 6. What is idempotent processing?

**Model answer:** Reprocessing same input does not duplicate/corrupt target.

**Follow-up:** Give a concrete example from Reliable Pipeline Simulator; state what evidence proves your answer.

## 7. What is atomic write?

**Model answer:** Output becomes complete unit via transaction or temp file+rename.

**Follow-up:** Give a concrete example from Reliable Pipeline Simulator; state what evidence proves your answer.

## 8. What is dead-letter queue?

**Model answer:** Separate destination for records repeatedly failing, with reason/replay metadata.

**Follow-up:** Give a concrete example from Reliable Pipeline Simulator; state what evidence proves your answer.

## 9. What is checkpoint recovery?

**Model answer:** Restart from last safe durable point and replay uncertain interval idempotently.

**Follow-up:** Give a concrete example from Reliable Pipeline Simulator; state what evidence proves your answer.

## 10. Why avoid exactly-once claim?

**Model answer:** Failures can occur between side effect/ack; state actual guarantees and dedup mechanism.

**Follow-up:** Give a concrete example from Reliable Pipeline Simulator; state what evidence proves your answer.

## 11. How handle bad row?

**Model answer:** Reject with reason and continue if safe, or fail batch for critical constraints; count it.

**Follow-up:** Give a concrete example from Reliable Pipeline Simulator; state what evidence proves your answer.

## 12. What is poison message?

**Model answer:** Message repeatedly failing due content/bug and blocking progress.

**Follow-up:** Give a concrete example from Reliable Pipeline Simulator; state what evidence proves your answer.

## 13. What is circuit breaker?

**Model answer:** Stops calls temporarily after failure threshold to allow dependency recovery.

**Follow-up:** Give a concrete example from Reliable Pipeline Simulator; state what evidence proves your answer.

## 14. How test recovery?

**Model answer:** Inject timeout, malformed record, crash before/after commit, duplicate and checkpoint failure.

**Follow-up:** Give a concrete example from Reliable Pipeline Simulator; state what evidence proves your answer.

## 15. Why preserve raw input?

**Model answer:** Supports replay/debug/audit/correction.

**Follow-up:** Give a concrete example from Reliable Pipeline Simulator; state what evidence proves your answer.

## 16. What is failure blast radius?

**Model answer:** Extent of downstream impact; isolate stages/partitions/transactions.

**Follow-up:** Give a concrete example from Reliable Pipeline Simulator; state what evidence proves your answer.

## 17. How handle partial output?

**Model answer:** Stage/temp output then validate and atomically publish/commit.

**Follow-up:** Give a concrete example from Reliable Pipeline Simulator; state what evidence proves your answer.

## 18. What is retry storm?

**Model answer:** Clients retry simultaneously, overload recovery; backoff/jitter/rate limit help.

**Follow-up:** Give a concrete example from Reliable Pipeline Simulator; state what evidence proves your answer.

## 19. How prevent retry duplicates?

**Model answer:** Stable event/business key, unique constraint, idempotency token or dedup ledger.

**Follow-up:** Give a concrete example from Reliable Pipeline Simulator; state what evidence proves your answer.

## 20. Defend reliability simulator.

**Model answer:** Describe injected failures, retry, durable checkpoint, idempotent output, quarantine and recovery tests.

**Follow-up:** Give a concrete example from Reliable Pipeline Simulator; state what evidence proves your answer.


---

# 2. Scenario questions

## 21. Failure after target commit?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 22. Poison row blocks batch?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 23. Design retry budget?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 24. Test atomic publication?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 25. Retryable vs permanent?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.


---

# 3. Project defence — Reliable Pipeline Simulator

## 90-second project answer

“My project is Reliable Pipeline Simulator. It solves this business problem: Pipeline may crash after reading, partially writing or committing before checkpoint. Design replay to preserve correctness. I define the source contract and record identity, read data in controlled units, validate structure/business rules, process valid records, isolate invalid data, persist safely and verify counts/keys/values. I record run metadata and progress only after durable success. I test duplicates, boundary cases and failures so reruns are safe. For production I would add managed secrets, least-privilege access, automated tests, scheduling, monitoring/alerts, scale tests and a runbook.”

## 1. Why choose this project?

**Answer:** It models a recurring Data Engineering responsibility and demonstrates more than a happy-path script.

## 2. What is the source contract?

**Answer:** Documented fields, types, required values, identity, delivery behavior and change expectations; I validate before processing.

## 3. How prove output correctness?

**Answer:** Reconcile counts, accepted/rejected totals, unique keys, business totals and source boundary.

## 4. What if job fails halfway?

**Answer:** Inspect last durable safe point, replay uncertain input idempotently, reconcile, then advance progress.

## 5. How prevent duplicates?

**Answer:** Stable business/event key, unique constraint or idempotency token and deterministic deduplication.

## 6. Largest limitation?

**Answer:** Classroom version uses small data and simplified infrastructure; production needs scale tests, secret management, concurrency and monitoring.

## 7. First improvement?

**Answer:** Automated boundary/failure tests, structured run metadata, alerting and repeatable deployment/runbook.

---

# 4. Practical interview exercise

- Draw source, validation, processing, destination, rejected path, checkpoint and monitoring. Label when progress becomes safe.
- Job processed 800 records, persisted 700, then crashed. Explain state inspection, retry, duplicate prevention and success checks.
- Compare classroom and production: scale, security, concurrency, retries, contract, observability and cost.
- Identify in your code: validation, error handling, persistence, progress and correctness verification. If absent, state limitation and next step.

# 5. Interview traps

- “HTTP 200 means correct data.”
- “I retry every error.”
- “I use timestamp” without ties/late records/deletes.
- “I remove duplicates” without identity key.
- “Job succeeded” without counts/reconciliation.
- “Exactly once” without mechanism/failure window.
- “I log everything” including secrets/PII.
- “Production-ready” based only on a local happy-path run.

# 6. Short revision answers

- **What is pipeline reliability?** Consistently produces correct, complete, recoverable results within expected operational limits.
- **Transient versus permanent?** Transient may recover (timeout/503); permanent needs correction (schema/credentials/bad request).
- **What is retry policy?** Eligible errors, attempts, delays, jitter, timeout and terminal behavior.
- **Why bounded retries?** Infinite retries consume resources and hide outages; bound and alert.
- **What is exponential backoff?** Increasing delay, often with jitter, to reduce pressure.
- **What is idempotent processing?** Reprocessing same input does not duplicate/corrupt target.
- **What is atomic write?** Output becomes complete unit via transaction or temp file+rename.
- **What is dead-letter queue?** Separate destination for records repeatedly failing, with reason/replay metadata.

# 7. Readiness checklist

- [ ] Core answers explained without reading.
- [ ] One business example connected.
- [ ] End-to-end flow drawn.
- [ ] Failure and recovery explained.
- [ ] Key/checkpoint choice defended.
- [ ] Correctness checks named.
- [ ] Limitations stated honestly.
- [ ] Production improvements discussed.

**Interview goal:** explain decision + reason + failure mode + verification, not just tool name.
