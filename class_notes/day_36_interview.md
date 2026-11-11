# Day 36 Interview Preparation: Batch Processing and Partitioning

**Mini-project:** Partitioned Batch Processor  
**Target:** Fresher / Junior Data Engineer  
**Method:** Answer aloud first; compare with the model answer; then connect it to your project.

---

# 1. Core concepts — complete model answers

## 1. What is batch processing?

**Model answer:** Processing finite records/files together on schedule or trigger.

**Follow-up:** Give a concrete example from Partitioned Batch Processor; state what evidence proves your answer.

## 2. What is batch size?

**Model answer:** Amount per unit; affects memory, throughput, overhead and recovery.

**Follow-up:** Give a concrete example from Partitioned Batch Processor; state what evidence proves your answer.

## 3. What is partitioning?

**Model answer:** Dividing data by key, often date, for storage and query scans.

**Follow-up:** Give a concrete example from Partitioned Batch Processor; state what evidence proves your answer.

## 4. What is partition pruning?

**Model answer:** Engine skips partitions impossible to match, reducing scanned data.

**Follow-up:** Give a concrete example from Partitioned Batch Processor; state what evidence proves your answer.

## 5. Why partition by date?

**Model answer:** Common filters and lifecycle align to dates; not always optimal.

**Follow-up:** Give a concrete example from Partitioned Batch Processor; state what evidence proves your answer.

## 6. What is Parquet?

**Model answer:** Typed columnar storage, compression and efficient analytical reads.

**Follow-up:** Give a concrete example from Partitioned Batch Processor; state what evidence proves your answer.

## 7. CSV versus Parquet?

**Model answer:** CSV simple text/compatible; Parquet typed columnar and often efficient but needs compatible tools.

**Follow-up:** Give a concrete example from Partitioned Batch Processor; state what evidence proves your answer.

## 8. What is small-file problem?

**Model answer:** Tiny files cause listing/metadata/scheduling/open overhead; compaction helps.

**Follow-up:** Give a concrete example from Partitioned Batch Processor; state what evidence proves your answer.

## 9. What is skew?

**Model answer:** Uneven partition data/work causes stragglers and resource imbalance.

**Follow-up:** Give a concrete example from Partitioned Batch Processor; state what evidence proves your answer.

## 10. How choose partition key?

**Model answer:** Common query filters, manageable cardinality, distribution and retention.

**Follow-up:** Give a concrete example from Partitioned Batch Processor; state what evidence proves your answer.

## 11. Why not customer partitions?

**Model answer:** High cardinality produces huge number of partitions/tiny files.

**Follow-up:** Give a concrete example from Partitioned Batch Processor; state what evidence proves your answer.

## 12. Explain pruning example.

**Model answer:** Date filter can scan only date partition if layout/engine supports it.

**Follow-up:** Give a concrete example from Partitioned Batch Processor; state what evidence proves your answer.

## 13. How handle timezone?

**Model answer:** Define UTC/business timezone before date partitioning.

**Follow-up:** Give a concrete example from Partitioned Batch Processor; state what evidence proves your answer.

## 14. How choose batch size?

**Model answer:** Measure memory, row width, overhead and throughput; tune from metrics.

**Follow-up:** Give a concrete example from Partitioned Batch Processor; state what evidence proves your answer.

## 15. What is parallelism?

**Model answer:** Concurrent processing of independent batches bounded by resources/source/target.

**Follow-up:** Give a concrete example from Partitioned Batch Processor; state what evidence proves your answer.

## 16. Can partitioning slow?

**Model answer:** Too many partitions/files or skew add overhead.

**Follow-up:** Give a concrete example from Partitioned Batch Processor; state what evidence proves your answer.

## 17. How compact files?

**Model answer:** Read small files per partition and write fewer sized files; verify schema/counts.

**Follow-up:** Give a concrete example from Partitioned Batch Processor; state what evidence proves your answer.

## 18. How validate partitions?

**Model answer:** Counts per partition, date boundaries, duplicates and file sizes.

**Follow-up:** Give a concrete example from Partitioned Batch Processor; state what evidence proves your answer.

## 19. Pruning versus filtering?

**Model answer:** Filtering selects rows; pruning avoids reading irrelevant partitions.

**Follow-up:** Give a concrete example from Partitioned Batch Processor; state what evidence proves your answer.

## 20. Defend partition processor.

**Model answer:** Explain key, batch size, file format, naming, compaction, pruning and reconciliation.

**Follow-up:** Give a concrete example from Partitioned Batch Processor; state what evidence proves your answer.


---

# 2. Scenario questions

## 21. Choose file size?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 22. What causes skew?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 23. How demonstrate pruning?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 24. How compact a partition?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 25. What if partition is empty?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.


---

# 3. Project defence — Partitioned Batch Processor

## 90-second project answer

“My project is Partitioned Batch Processor. It solves this business problem: A 500 GB dataset needs bounded processing and query-aware partition layout; avoid memory blowups and unnecessary scans. I define the source contract and record identity, read data in controlled units, validate structure/business rules, process valid records, isolate invalid data, persist safely and verify counts/keys/values. I record run metadata and progress only after durable success. I test duplicates, boundary cases and failures so reruns are safe. For production I would add managed secrets, least-privilege access, automated tests, scheduling, monitoring/alerts, scale tests and a runbook.”

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

- **What is batch processing?** Processing finite records/files together on schedule or trigger.
- **What is batch size?** Amount per unit; affects memory, throughput, overhead and recovery.
- **What is partitioning?** Dividing data by key, often date, for storage and query scans.
- **What is partition pruning?** Engine skips partitions impossible to match, reducing scanned data.
- **Why partition by date?** Common filters and lifecycle align to dates; not always optimal.
- **What is Parquet?** Typed columnar storage, compression and efficient analytical reads.
- **CSV versus Parquet?** CSV simple text/compatible; Parquet typed columnar and often efficient but needs compatible tools.
- **What is small-file problem?** Tiny files cause listing/metadata/scheduling/open overhead; compaction helps.

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
