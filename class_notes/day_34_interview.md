# Day 34 Interview Preparation: Data Transformation and Business Rules

**Mini-project:** Transformation Pipeline  
**Target:** Fresher / Junior Data Engineer  
**Method:** Answer aloud first; compare with the model answer; then connect it to your project.

---

# 1. Core concepts — complete model answers

## 1. What is transformation?

**Model answer:** Applying explicit rules to convert raw data into consistent target representation: types, mappings, derived values and business logic.

**Follow-up:** Give a concrete example from Transformation Pipeline; state what evidence proves your answer.

## 2. Cleaning versus transformation?

**Model answer:** Cleaning handles quality problems; transformation is broader and changes structure/meaning; cleaning is one stage.

**Follow-up:** Give a concrete example from Transformation Pipeline; state what evidence proves your answer.

## 3. What is standardization?

**Model answer:** Making equivalent representations consistent, e.g. trimming whitespace or approved casing.

**Follow-up:** Give a concrete example from Transformation Pipeline; state what evidence proves your answer.

## 4. Why document business rules?

**Model answer:** Rules define business meaning; documentation enables review, testing, reproducibility and domain agreement.

**Follow-up:** Give a concrete example from Transformation Pipeline; state what evidence proves your answer.

## 5. How handle unknown categories?

**Model answer:** Preserve original value, flag/quarantine and alert; do not silently discard or guess.

**Follow-up:** Give a concrete example from Transformation Pipeline; state what evidence proves your answer.

## 6. Why distinguish NULL and zero?

**Model answer:** NULL means unknown/not provided; zero is known numeric. Replacing null distorts totals.

**Follow-up:** Give a concrete example from Transformation Pipeline; state what evidence proves your answer.

## 7. What is derived column?

**Model answer:** Calculated from fields, e.g. quantity×unit_price; formula, precision and null behavior must be explicit.

**Follow-up:** Give a concrete example from Transformation Pipeline; state what evidence proves your answer.

## 8. Why transformation order?

**Model answer:** Parse types before calculation, validate inputs before deriving and apply rules in documented order.

**Follow-up:** Give a concrete example from Transformation Pipeline; state what evidence proves your answer.

## 9. How handle money?

**Model answer:** Use Decimal/database decimal; avoid binary float for exact currency and round at approved stage.

**Follow-up:** Give a concrete example from Transformation Pipeline; state what evidence proves your answer.

## 10. What is lineage?

**Model answer:** Metadata linking output to source records/files, runs and transformation versions.

**Follow-up:** Give a concrete example from Transformation Pipeline; state what evidence proves your answer.

## 11. How test transformation?

**Model answer:** Representative valid/invalid/boundary/null/unknown cases, expected outputs and reconciliation invariants.

**Follow-up:** Give a concrete example from Transformation Pipeline; state what evidence proves your answer.

## 12. What is mapping table?

**Model answer:** Maintained translation of source codes to target codes; easier to review/update than scattered hard-coded conditions.

**Follow-up:** Give a concrete example from Transformation Pipeline; state what evidence proves your answer.

## 13. Why preserve raw data?

**Model answer:** Supports replay, audit, comparison and correction when rules change.

**Follow-up:** Give a concrete example from Transformation Pipeline; state what evidence proves your answer.

## 14. What is reject record?

**Model answer:** Source row retained with reason, source location and run ID for failed validation.

**Follow-up:** Give a concrete example from Transformation Pipeline; state what evidence proves your answer.

## 15. What is quality invariant?

**Model answer:** Condition expected to hold, e.g. quantity>0 or input=accepted+rejected.

**Follow-up:** Give a concrete example from Transformation Pipeline; state what evidence proves your answer.

## 16. How prevent silent loss?

**Model answer:** Count accepted/rejected, report reasons and alert thresholds.

**Follow-up:** Give a concrete example from Transformation Pipeline; state what evidence proves your answer.

## 17. What is rule versioning?

**Model answer:** Record code/rule version so output can be reproduced.

**Follow-up:** Give a concrete example from Transformation Pipeline; state what evidence proves your answer.

## 18. What is aggregation?

**Model answer:** Combining records into summaries; validate grouping keys and duplicates to avoid inflated totals.

**Follow-up:** Give a concrete example from Transformation Pipeline; state what evidence proves your answer.

## 19. Explain status normalization.

**Model answer:** Trim, normalize approved case variants, route unknown statuses to exception rather than inventing meaning.

**Follow-up:** Give a concrete example from Transformation Pipeline; state what evidence proves your answer.

## 20. Defend transformation project.

**Model answer:** Describe raw input, rules, conversions, rejects, validation, output, tests and reconciliation.

**Follow-up:** Give a concrete example from Transformation Pipeline; state what evidence proves your answer.


---

# 2. Scenario questions

## 21. What if quantity is negative?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 22. How map source codes?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 23. When reject a row?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 24. How test rounding?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 25. Explain lineage to auditor.

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.


---

# 3. Project defence — Transformation Pipeline

## 90-second project answer

“My project is Transformation Pipeline. It solves this business problem: Raw orders contain inconsistent statuses and types. Standardize them without silently changing meaning. I define the source contract and record identity, read data in controlled units, validate structure/business rules, process valid records, isolate invalid data, persist safely and verify counts/keys/values. I record run metadata and progress only after durable success. I test duplicates, boundary cases and failures so reruns are safe. For production I would add managed secrets, least-privilege access, automated tests, scheduling, monitoring/alerts, scale tests and a runbook.”

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

- **What is transformation?** Applying explicit rules to convert raw data into consistent target representation: types, mappings, derived values and business logic.
- **Cleaning versus transformation?** Cleaning handles quality problems; transformation is broader and changes structure/meaning; cleaning is one stage.
- **What is standardization?** Making equivalent representations consistent, e.g. trimming whitespace or approved casing.
- **Why document business rules?** Rules define business meaning; documentation enables review, testing, reproducibility and domain agreement.
- **How handle unknown categories?** Preserve original value, flag/quarantine and alert; do not silently discard or guess.
- **Why distinguish NULL and zero?** NULL means unknown/not provided; zero is known numeric. Replacing null distorts totals.
- **What is derived column?** Calculated from fields, e.g. quantity×unit_price; formula, precision and null behavior must be explicit.
- **Why transformation order?** Parse types before calculation, validate inputs before deriving and apply rules in documented order.

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
