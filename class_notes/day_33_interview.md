# Day 33 Interview Preparation: Database Ingestion

**Mini-project:** Database Extractor  
**Target:** Fresher / Junior Data Engineer  
**Method:** Answer aloud first; compare with the model answer; then connect it to your project.

---

# 1. Core concepts — complete model answers

## 1. What is database ingestion?

**Model answer:** Moving source database records into staging/analytics with controlled load, consistency and progress.

**Follow-up:** Give a concrete example from Database Extractor; state what evidence proves your answer.

## 2. Why use a read-only account?

**Model answer:** It limits accidental writes and security exposure. Grant only needed SELECT permissions.

**Follow-up:** Give a concrete example from Database Extractor; state what evidence proves your answer.

## 3. What is a cursor?

**Model answer:** Driver object for SQL execution and row fetching; distinct from API cursor.

**Follow-up:** Give a concrete example from Database Extractor; state what evidence proves your answer.

## 4. fetchall versus fetchmany?

**Model answer:** fetchall loads all rows in memory; fetchmany returns bounded groups. Large extracts need batch/server-side cursor/key ranges.

**Follow-up:** Give a concrete example from Database Extractor; state what evidence proves your answer.

## 5. Why explicit columns?

**Model answer:** Avoids unnecessary transfer and makes schema changes visible; SELECT * may include sensitive/large columns.

**Follow-up:** Give a concrete example from Database Extractor; state what evidence proves your answer.

## 6. What is parameterized SQL?

**Model answer:** Placeholders separate values from SQL text, prevent injection and handle quoting/types safely.

**Follow-up:** Give a concrete example from Database Extractor; state what evidence proves your answer.

## 7. How extract incrementally?

**Model answer:** Use reliable change column/CDC plus saved boundary, deterministic ordering, overlap if needed; persist then checkpoint.

**Follow-up:** Give a concrete example from Database Extractor; state what evidence proves your answer.

## 8. Why ID-only extraction can fail?

**Model answer:** Higher IDs find inserts but miss updates/deletes to old IDs.

**Follow-up:** Give a concrete example from Database Extractor; state what evidence proves your answer.

## 9. What is CDC?

**Model answer:** Change Data Capture records inserts/updates/deletes from logs/events instead of rescanning.

**Follow-up:** Give a concrete example from Database Extractor; state what evidence proves your answer.

## 10. What is source consistency?

**Model answer:** Extract should represent coherent view; concurrent updates can create inconsistencies without snapshot/CDC strategy.

**Follow-up:** Give a concrete example from Database Extractor; state what evidence proves your answer.

## 11. What is a read replica?

**Model answer:** Replica serves reads but replication lag may omit newest source changes.

**Follow-up:** Give a concrete example from Database Extractor; state what evidence proves your answer.

## 12. Why ORDER BY?

**Model answer:** SQL has no guaranteed order without ORDER BY; deterministic order supports batches/checkpoints.

**Follow-up:** Give a concrete example from Database Extractor; state what evidence proves your answer.

## 13. What is a watermark tie?

**Model answer:** Multiple rows share timestamp; timestamp-only boundary may skip ties. Compound timestamp+key or overlap solves.

**Follow-up:** Give a concrete example from Database Extractor; state what evidence proves your answer.

## 14. How extract millions of rows?

**Model answer:** Select only needed columns, filter source-side, fetch bounded batches, use CDC/key ranges and monitor source load.

**Follow-up:** Give a concrete example from Database Extractor; state what evidence proves your answer.

## 15. What does connection context do?

**Model answer:** It manages resource lifecycle and transaction behavior on exit; understand driver-specific commit/rollback semantics.

**Follow-up:** Give a concrete example from Database Extractor; state what evidence proves your answer.

## 16. Why avoid long transactions?

**Model answer:** They can retain locks/snapshots/resources and affect source operations.

**Follow-up:** Give a concrete example from Database Extractor; state what evidence proves your answer.

## 17. How capture deletes?

**Model answer:** Use CDC deletes, tombstones, soft-delete fields or periodic reconciliation; SELECT cannot see deleted rows.

**Follow-up:** Give a concrete example from Database Extractor; state what evidence proves your answer.

## 18. How verify extraction?

**Model answer:** Counts, key sets/ranges, aggregates, rejects and watermarks across source/target.

**Follow-up:** Give a concrete example from Database Extractor; state what evidence proves your answer.

## 19. What is key-range extraction?

**Model answer:** Bound batches with ID > last and ID <= upper bound; suitable append-only but updates/deletes need another strategy.

**Follow-up:** Give a concrete example from Database Extractor; state what evidence proves your answer.

## 20. Defend database extractor.

**Model answer:** Least privilege, explicit SQL, batch fetch, validation, safe write/checkpoint, failure handling and reconciliation.

**Follow-up:** Give a concrete example from Database Extractor; state what evidence proves your answer.


---

# 2. Scenario questions

## 21. How avoid OLTP impact?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 22. What if updated_at is null?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 23. How handle timezone?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 24. What is consistent snapshot?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 25. Which metrics prove completeness?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.


---

# 3. Project defence — Database Extractor

## 90-second project answer

“My project is Database Extractor. It solves this business problem: Copy operational data without overwhelming source DB; extract with stable progress and reconcile source/target. I define the source contract and record identity, read data in controlled units, validate structure/business rules, process valid records, isolate invalid data, persist safely and verify counts/keys/values. I record run metadata and progress only after durable success. I test duplicates, boundary cases and failures so reruns are safe. For production I would add managed secrets, least-privilege access, automated tests, scheduling, monitoring/alerts, scale tests and a runbook.”

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

- **What is database ingestion?** Moving source database records into staging/analytics with controlled load, consistency and progress.
- **Why use a read-only account?** It limits accidental writes and security exposure. Grant only needed SELECT permissions.
- **What is a cursor?** Driver object for SQL execution and row fetching; distinct from API cursor.
- **fetchall versus fetchmany?** fetchall loads all rows in memory; fetchmany returns bounded groups. Large extracts need batch/server-side cursor/key ranges.
- **Why explicit columns?** Avoids unnecessary transfer and makes schema changes visible; SELECT * may include sensitive/large columns.
- **What is parameterized SQL?** Placeholders separate values from SQL text, prevent injection and handle quoting/types safely.
- **How extract incrementally?** Use reliable change column/CDC plus saved boundary, deterministic ordering, overlap if needed; persist then checkpoint.
- **Why ID-only extraction can fail?** Higher IDs find inserts but miss updates/deletes to old IDs.

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
