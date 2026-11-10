# Day 5 — Data Pipeline Fundamentals | Interview Preparation

## Core Q&A

### 1. Define a data pipeline.
A repeatable process that ingests data, stores/transforms/validates it, and delivers it to a destination with operational controls.

### 2. What are pipeline lifecycle stages?
Requirements → source discovery → design → development → testing/validation → deployment → monitoring/operations → improvement or retirement.

### 3. Reliability vs scalability vs fault tolerance?
- Reliability: consistently correct operation.
- Scalability: capacity to handle growth.
- Fault tolerance: ability to continue or recover when components fail.

### 4. What is idempotency?
Repeating the same operation produces the same intended final state without unintended duplicates or extra effects. It is essential for safe retries and reruns.

### 5. Why are checkpoints/watermarks used?
They track progress in incremental processing so the next run can continue from a known point. They must be advanced only after output is safely committed and validated.

### 6. What is data governance?
Policies and responsibilities for data ownership, definitions, quality, access, lineage, retention, and compliant use.

### 7. How do you secure a pipeline?
Least-privilege identities, secrets management, encrypted connections/storage as required, restricted raw access, safe logging, environment separation, and credential rotation.

### 8. What should pipeline monitoring include?
Run status, duration, source/destination counts, rejected records, freshness, retry/error counts, and alerts tied to an owner.

## Scenario: pipeline writes partial data then crashes
1. Determine what was committed.
2. Avoid blindly rerunning append logic.
3. Use transactional/staging or idempotent merge strategy.
4. Resume from a safe checkpoint or replace the affected partition.
5. Reconcile counts and key metrics.
6. Advance watermark only after successful commit/validation.
7. Add a failure test.

## Scenario: source schema changes unexpectedly
- Detect schema mismatch.
- Stop or quarantine unsafe data rather than silently corrupting it.
- Preserve raw records.
- Notify owner and assess downstream impact.
- Update contract/mapping and tests.
- Deploy a compatible change and backfill if needed.

## Scenario: pipeline is successful but dashboard is stale
Check data freshness and publication time, not only task status. Inspect source arrival, extraction watermark, transformation dependencies, failed partitions, and consumer cache/refresh.

## Interview design framework
When asked to design a pipeline, cover:
1. Business metric and SLA.
2. Sources, keys, volume, change pattern.
3. Batch/stream choice.
4. Raw and curated storage.
5. Transformation and data model.
6. Quality and reconciliation.
7. Incremental/idempotent processing.
8. Security and governance.
9. Orchestration, monitoring, alerting.
10. Recovery, backfill, cost, ownership.

## Quick revision
A strong answer always includes correctness, safe reruns, quality gates, security, observability, and business freshness—not only tools.
