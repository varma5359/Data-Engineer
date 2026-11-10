# Day 5 Interview Preparation: Data Pipeline Fundamentals

**Mini-project:** Pipeline Requirements Document

## 1. Core questions

### 1. What makes a data pipeline reliable?
Correct extraction, validation, duplicate-safe processing, safe retries, checkpointing, recovery, monitoring, clear ownership and reconciliation.

### 2. What is idempotency?
Repeating an operation with the same input produces the same final state as running it once. It prevents retries from creating duplicate business effects.

### 3. What is checkpointing?
Persisting processing progress so a pipeline can resume from a known safe point after interruption.

### 4. What is fault tolerance?
The ability to continue operating or recover when a component fails, using techniques such as retries, replication, durable storage and recovery procedures.

### 5. What is scalability?
The ability to handle increased data volume, frequency or concurrency while meeting performance and reliability requirements.

### 6. What is data governance?
The policies and practices that define data ownership, meaning, quality, access, protection, lineage and retention.

### 7. What is least privilege?
Giving each person or service only the permissions required to perform its task.

## 2. Difference-based questions

| Compare | Strong answer |
|---|---|
| Retry vs checkpoint | Retry repeats an operation; checkpoint records progress |
| Reliability vs availability | Reliability concerns correct consistent operation; availability concerns being accessible when needed |
| RTO vs RPO | Target recovery time vs acceptable recovery point/data loss |
| Transient vs permanent failure | Temporary service issue vs error requiring correction |
| Full vs incremental load | Complete dataset vs new/changed records |
| Logs vs metrics | Detailed event records vs numerical measurements over time |

## 3. Practical implementation questions

### 8. How do you make a load safe to retry?
Use stable business keys, stage data before applying changes, perform a merge/upsert or equivalent atomic operation, record batch identity and reconcile the target after execution.

### 9. What should a pipeline log?
Run ID, pipeline name, source and target, timestamps, status, rows read/written/rejected, duration and a safe error summary. Never log secrets.

### 10. How would you recover from a failed incremental load?
Determine the last committed checkpoint, identify whether writes were committed, inspect the failed batch, correct the issue, replay from a safe boundary and reconcile source and target.

### 11. How do you scale a pipeline?
Measure the bottleneck, then consider incremental extraction, partitioning, parallelism, efficient formats, query pushdown and resource sizing. Validate correctness and cost after changes.

## 4. Scenario questions

### Scenario A: A pipeline ran twice
**Problem:** The scheduler retried a successful but unacknowledged load.

**Action:** Use batch IDs and stable record keys with an idempotent merge. Check target counts and totals after the second run.

**Reasoning:** A retry should not duplicate business data.

### Scenario B: Source schema changed overnight
**Problem:** A required field changed from integer to text.

**Action:** Stop or quarantine incompatible input, compare with the source contract, contact the owner, assess downstream impact, update parsing and tests, then resume after validation.

**Reasoning:** Silent coercion can corrupt keys and measures.

### Scenario C: Dashboard is late
**Problem:** A daily pipeline misses its 7 AM target.

**Action:** Inspect stage-level duration, source arrival, retries, queue time, warehouse load and dependencies. Identify the bottleneck and communicate the impact.

**Reasoning:** Optimize based on evidence rather than increasing resources blindly.

## 5. Troubleshooting

### 12. Row counts differ between source and target
Compare extraction windows, filters, keys, duplicates, late updates, rejected rows and commit status. Reconcile by key and business measure.

### 13. Pipeline memory usage grows
Inspect whether the process loads all data at once, whether batches are too large, whether objects are retained, and whether partitioned or streaming reads are suitable.

### 14. Retries create duplicate records
Inspect write semantics and keys. Replace append-only behavior with an idempotent strategy and test repeated execution.

### 15. Alerts are noisy
Set meaningful thresholds, distinguish warning from failure, suppress duplicate notifications and assign ownership.

## 6. Follow-up questions

- What is a dead-letter queue?
- What is a backfill?
- What is a data contract?
- What is lineage?
- How do you handle late-arriving records?
- What is a recovery runbook?
- Why measure freshness?
- How do you test a recovery procedure?

## 7. Strong revision points

- Reliability is about correct data, not merely successful execution.
- Make retries safe before enabling automatic retries.
- Commit work before advancing a checkpoint.
- Reconcile counts and business measures.
- Monitor freshness, duration, throughput and rejected records.
- Define ownership and recovery responsibilities.

## 8. Mini-project defence

Present your Pipeline Requirements Document as if handing it to an engineering team.

Be ready to defend:

1. Business objective and consumers.
2. Source/target contract and key.
3. Incremental extraction strategy.
4. Data quality checks.
5. Duplicate-safe loading.
6. Recovery and checkpoint rules.
7. Security and access controls.
8. Freshness and availability targets.
9. Monitoring and alert ownership.

**Strong closing principle:** “I would not call the pipeline production-ready until its correctness checks, failure behavior, recovery process and operational ownership have been tested.”
