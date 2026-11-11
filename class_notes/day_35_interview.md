# Day 35 Interview Preparation: Data Cleaning and Schema Evolution

**Mini-project:** Data Cleaning Framework  
**Target:** Fresher / Junior Data Engineer  
**Method:** Answer aloud first; compare with the model answer; then connect it to your project.

---

# 1. Core concepts — complete model answers

## 1. What is schema drift?

**Model answer:** Unexpected incoming structure change, such as rename/removal/type change.

**Follow-up:** Give a concrete example from Data Cleaning Framework; state what evidence proves your answer.

## 2. What is schema evolution?

**Model answer:** Controlled, versioned schema change with compatibility and consumer impact assessed.

**Follow-up:** Give a concrete example from Data Cleaning Framework; state what evidence proves your answer.

## 3. What is data contract?

**Model answer:** Agreement on fields, types, semantics, quality, delivery, ownership and change process.

**Follow-up:** Give a concrete example from Data Cleaning Framework; state what evidence proves your answer.

## 4. How detect schema drift?

**Model answer:** Compare received columns/types/nullability/version with approved contract.

**Follow-up:** Give a concrete example from Data Cleaning Framework; state what evidence proves your answer.

## 5. What is additive change?

**Model answer:** Adding a field; compatible only if consumers tolerate it and field is optional/defaulted.

**Follow-up:** Give a concrete example from Data Cleaning Framework; state what evidence proves your answer.

## 6. Why rename is breaking?

**Model answer:** Consumers expect old name; rename appears as missing plus extra and breaks mapping.

**Follow-up:** Give a concrete example from Data Cleaning Framework; state what evidence proves your answer.

## 7. What is backward compatibility?

**Model answer:** New version remains usable by older consumers under agreed rules.

**Follow-up:** Give a concrete example from Data Cleaning Framework; state what evidence proves your answer.

## 8. How define duplicates?

**Model answer:** Use business/event identity, not all columns; identical payloads can be legitimate separate events.

**Follow-up:** Give a concrete example from Data Cleaning Framework; state what evidence proves your answer.

## 9. How handle missing values?

**Model answer:** Field-level policy: preserve NULL, meaningful default, reject required missing, or approved imputation.

**Follow-up:** Give a concrete example from Data Cleaning Framework; state what evidence proves your answer.

## 10. Why profile data?

**Model answer:** Measure nulls, distinct values, ranges, duplicates and distributions before cleaning.

**Follow-up:** Give a concrete example from Data Cleaning Framework; state what evidence proves your answer.

## 11. What is quarantine?

**Model answer:** Controlled holding area preserving evidence/reason for investigation and replay.

**Follow-up:** Give a concrete example from Data Cleaning Framework; state what evidence proves your answer.

## 12. What is schema registry?

**Model answer:** Central schema/version and compatibility rules, often for event/message formats.

**Follow-up:** Give a concrete example from Data Cleaning Framework; state what evidence proves your answer.

## 13. How handle type change?

**Model answer:** Assess semantics/consumers, define migration/cast, version contract and test both; no silent arbitrary cast.

**Follow-up:** Give a concrete example from Data Cleaning Framework; state what evidence proves your answer.

## 14. Should unknown columns reject?

**Model answer:** Capture and alert; tolerate additive fields only when contract/consumer allow.

**Follow-up:** Give a concrete example from Data Cleaning Framework; state what evidence proves your answer.

## 15. What is idempotent dedup?

**Model answer:** Repeated application yields same result; identity and winner rule deterministic.

**Follow-up:** Give a concrete example from Data Cleaning Framework; state what evidence proves your answer.

## 16. What is quality rule?

**Model answer:** Not-null, uniqueness, range, referential integrity or accepted values.

**Follow-up:** Give a concrete example from Data Cleaning Framework; state what evidence proves your answer.

## 17. How clean phone numbers?

**Model answer:** Normalize with country/context rules; retain original and avoid assumptions.

**Follow-up:** Give a concrete example from Data Cleaning Framework; state what evidence proves your answer.

## 18. What is schema version?

**Model answer:** Identifier for structure/meaning version enabling controlled migration.

**Follow-up:** Give a concrete example from Data Cleaning Framework; state what evidence proves your answer.

## 19. How test evolution?

**Model answer:** Test old/new fixtures, optional/additive fields and breaking changes downstream.

**Follow-up:** Give a concrete example from Data Cleaning Framework; state what evidence proves your answer.

## 20. Defend cleaning framework.

**Model answer:** Explain contract checks, profiling, duplicate keys, null policy, quarantine, versions, metrics and replay.

**Follow-up:** Give a concrete example from Data Cleaning Framework; state what evidence proves your answer.


---

# 2. Scenario questions

## 21. What if required field disappears?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 22. How roll out rename?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 23. What is breaking change?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 24. How preserve unknown columns?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 25. How replay after mapping fix?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.


---

# 3. Project defence — Data Cleaning Framework

## 90-second project answer

“My project is Data Cleaning Framework. It solves this business problem: A supplier renames customer_id to customerId and adds a field. Detect and manage schema changes rather than silently misinterpreting them. I define the source contract and record identity, read data in controlled units, validate structure/business rules, process valid records, isolate invalid data, persist safely and verify counts/keys/values. I record run metadata and progress only after durable success. I test duplicates, boundary cases and failures so reruns are safe. For production I would add managed secrets, least-privilege access, automated tests, scheduling, monitoring/alerts, scale tests and a runbook.”

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

- **What is schema drift?** Unexpected incoming structure change, such as rename/removal/type change.
- **What is schema evolution?** Controlled, versioned schema change with compatibility and consumer impact assessed.
- **What is data contract?** Agreement on fields, types, semantics, quality, delivery, ownership and change process.
- **How detect schema drift?** Compare received columns/types/nullability/version with approved contract.
- **What is additive change?** Adding a field; compatible only if consumers tolerate it and field is optional/defaulted.
- **Why rename is breaking?** Consumers expect old name; rename appears as missing plus extra and breaks mapping.
- **What is backward compatibility?** New version remains usable by older consumers under agreed rules.
- **How define duplicates?** Use business/event identity, not all columns; identical payloads can be legitimate separate events.

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
