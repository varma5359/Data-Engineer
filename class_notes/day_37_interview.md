# Day 37 Interview Preparation: Incremental Loading and Upserts

**Mini-project:** Incremental Data Loader  
**Target:** Fresher / Junior Data Engineer  
**Method:** Answer aloud first; compare with the model answer; then connect it to your project.

---

# 1. Core concepts — complete model answers

## 1. What is incremental loading?

**Model answer:** Loading only new/changed/deleted source data since prior progress.

**Follow-up:** Give a concrete example from Incremental Data Loader; state what evidence proves your answer.

## 2. What is upsert?

**Model answer:** Insert if absent or update if present.

**Follow-up:** Give a concrete example from Incremental Data Loader; state what evidence proves your answer.

## 3. Why unique key?

**Model answer:** Defines identity and makes conflict/duplicate handling reliable.

**Follow-up:** Give a concrete example from Incremental Data Loader; state what evidence proves your answer.

## 4. What is watermark?

**Model answer:** Saved boundary like updated_at plus ID for extraction progress.

**Follow-up:** Give a concrete example from Incremental Data Loader; state what evidence proves your answer.

## 5. What is checkpoint?

**Model answer:** Durable state for completed progress, cursor/run/target status.

**Follow-up:** Give a concrete example from Incremental Data Loader; state what evidence proves your answer.

## 6. What is idempotency?

**Model answer:** Repeated replay yields correct final state without duplicate side effects.

**Follow-up:** Give a concrete example from Incremental Data Loader; state what evidence proves your answer.

## 7. What is PostgreSQL ON CONFLICT?

**Model answer:** PostgreSQL INSERT clause handles unique conflict by update or alternative.

**Follow-up:** Give a concrete example from Incremental Data Loader; state what evidence proves your answer.

## 8. What is EXCLUDED?

**Model answer:** Proposed incoming row in PostgreSQL ON CONFLICT DO UPDATE.

**Follow-up:** Give a concrete example from Incremental Data Loader; state what evidence proves your answer.

## 9. Why stage before upsert?

**Model answer:** Allows validation, duplicate checks, counts and transaction boundaries.

**Follow-up:** Give a concrete example from Incremental Data Loader; state what evidence proves your answer.

## 10. How handle late updates?

**Model answer:** CDC, overlap/lookback, reliable update timestamp or reconciliation.

**Follow-up:** Give a concrete example from Incremental Data Loader; state what evidence proves your answer.

## 11. How handle deletes?

**Model answer:** CDC delete events/tombstones, soft delete or source-target reconciliation.

**Follow-up:** Give a concrete example from Incremental Data Loader; state what evidence proves your answer.

## 12. What is MERGE?

**Model answer:** SQL construct to match source/target and insert/update/delete; engine syntax differs.

**Follow-up:** Give a concrete example from Incremental Data Loader; state what evidence proves your answer.

## 13. When full load useful?

**Model answer:** Small tables, initial/rebuild, or no reliable change capture.

**Follow-up:** Give a concrete example from Incremental Data Loader; state what evidence proves your answer.

## 14. Target commit but checkpoint fails?

**Model answer:** Replay may occur; idempotent upsert makes safe, then checkpoint can advance.

**Follow-up:** Give a concrete example from Incremental Data Loader; state what evidence proves your answer.

## 15. Checkpoint before commit?

**Model answer:** Unsafe: failure can skip data permanently.

**Follow-up:** Give a concrete example from Incremental Data Loader; state what evidence proves your answer.

## 16. What is lookback?

**Model answer:** Re-read recent interval for late commits/ties; deduplicate idempotently.

**Follow-up:** Give a concrete example from Incremental Data Loader; state what evidence proves your answer.

## 17. How reconcile?

**Model answer:** Counts, key sets, aggregates, boundaries and checksums as appropriate.

**Follow-up:** Give a concrete example from Incremental Data Loader; state what evidence proves your answer.

## 18. Why separate ingestion and source time?

**Model answer:** Source update time describes change; ingestion time says pipeline observed it.

**Follow-up:** Give a concrete example from Incremental Data Loader; state what evidence proves your answer.

## 19. What does exactly-once mean?

**Model answer:** Distributed delivery often replayed; use at-least-once plus idempotent effect and describe actual guarantee.

**Follow-up:** Give a concrete example from Incremental Data Loader; state what evidence proves your answer.

## 20. Defend incremental loader.

**Model answer:** Cursor, key, stage, upsert transaction, delete strategy, checkpoint order, replay and reconciliation.

**Follow-up:** Give a concrete example from Incremental Data Loader; state what evidence proves your answer.


---

# 2. Scenario questions

## 21. Two updates same timestamp?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 22. What if batch replayed?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 23. How handle deletes?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 24. Initial load then incremental?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 25. How audit target?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.


---

# 3. Project defence — Incremental Data Loader

## 90-second project answer

“My project is Incremental Data Loader. It solves this business problem: Target must insert new records and update changed ones without duplicates while retaining safe progress. I define the source contract and record identity, read data in controlled units, validate structure/business rules, process valid records, isolate invalid data, persist safely and verify counts/keys/values. I record run metadata and progress only after durable success. I test duplicates, boundary cases and failures so reruns are safe. For production I would add managed secrets, least-privilege access, automated tests, scheduling, monitoring/alerts, scale tests and a runbook.”

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

- **What is incremental loading?** Loading only new/changed/deleted source data since prior progress.
- **What is upsert?** Insert if absent or update if present.
- **Why unique key?** Defines identity and makes conflict/duplicate handling reliable.
- **What is watermark?** Saved boundary like updated_at plus ID for extraction progress.
- **What is checkpoint?** Durable state for completed progress, cursor/run/target status.
- **What is idempotency?** Repeated replay yields correct final state without duplicate side effects.
- **What is PostgreSQL ON CONFLICT?** PostgreSQL INSERT clause handles unique conflict by update or alternative.
- **What is EXCLUDED?** Proposed incoming row in PostgreSQL ON CONFLICT DO UPDATE.

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
