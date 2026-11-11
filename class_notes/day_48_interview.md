# Day 48 Interview Guide — Warehouse Loading Strategies

## 1. What is a full load?

A full load reads all selected source records and rebuilds or replaces the target. It is simple and useful for small reference datasets or controlled rebuilds, but can be expensive and risky for large tables.

## 2. What is append loading?

Append adds new rows without replacing existing rows. It is suitable for immutable event data when a stable event key and duplicate/replay protection exist. Replaying the same batch without protection creates duplicates.

## 3. What is overwrite loading?

Overwrite replaces an entire target or selected partition with a new result. It is useful for snapshots or rebuilding a corrected date partition. The replacement must be complete and safely published.

## 4. What is incremental loading?

Incremental loading processes only new or changed source records since a known progress point. It reduces repeated work but requires reliable change detection, checkpoint management, update/delete handling and replay safety.

## 5. What is an upsert?

An upsert inserts a row if its key does not exist and updates it if the key already exists. PostgreSQL supports ON CONFLICT. Upsert commonly maintains current-state tables but does not automatically preserve history.

## 6. Upsert versus SCD Type 2?

Upsert often overwrites the current target row. SCD Type 2 creates a new dimension version and preserves the old one. The business history requirement determines which is correct.

## 7. What is a watermark?

A watermark records source progress, such as the latest processed update timestamp or CDC sequence. It must be advanced only after the target write and required validation are durable.

## 8. Why can a timestamp watermark miss rows?

Multiple source rows can share a timestamp. If the query uses only greater-than the last timestamp, tied rows may be skipped. Late-arriving changes can also fall behind. Use a composite cursor, CDC position or a controlled overlap-and-deduplication strategy.

## 9. What is idempotency?

Idempotency means repeating the same logical input does not create unintended additional effects. It is important because jobs can fail after a destination write but before checkpoint acknowledgement. Stable keys, unique constraints, deterministic upserts and reconciliation help make replay safe.

## 10. Why use staging?

Staging separates ingestion from warehouse publication. It provides a place to validate schema, types, keys, duplicates and business rules before data is loaded into curated models.

## 11. How do you handle deletes?

First determine how the source represents deletes. Options include CDC delete events, soft-delete flags, periodic full comparison or tombstones. The target behavior must be documented because simply not receiving a row does not always mean it was deleted.

## 12. Where does dbt fit?

In a common ELT architecture, ingestion loads raw data into the warehouse. dbt then manages SQL transformations, dependencies, tests and documentation to build staging, intermediate and mart models. It does not replace every source extraction or CDC system.

## 13. How do you safely replace a partition?

Identify the exact affected partition, build the complete replacement separately, validate its counts and values, then publish it using a platform-supported safe replacement method. A partial source extract must not overwrite a complete partition.

## 14. Scenario: target write succeeded but checkpoint save failed.

The next run may replay the same input. The target write must be idempotent or deduplicated using stable keys. I would inspect the committed target, replay safely, reconcile and then advance the checkpoint.

## 15. Scenario: the source sends updates but the pipeline only appends.

The target will retain stale values or duplicate versions. I would use an upsert for current-state requirements or SCD Type 2 for historical requirements, based on the business rule.

## 16. Explain your Day 48 load simulator.

I created examples for full load, append and upsert, tested duplicate-key protection and documented an incremental strategy with staging validation, safe watermark handling, idempotent writes and reconciliation. I compared the strategy against different source behaviors.

## Interview traps

- Calling append idempotent without a deduplication mechanism.
- Treating upsert as history preservation.
- Advancing a watermark before the target is durable.
- Assuming timestamps are unique.
- Ignoring source deletes.
- Overwriting a partition before validating replacement data.
- Claiming dbt handles all ingestion.

## Readiness checklist

- [ ] Explain five loading strategies.
- [ ] Explain replay and idempotency.
- [ ] Explain watermark tie and late-arrival risks.
- [ ] Explain upsert versus SCD Type 2.
- [ ] Describe safe partition replacement.
- [ ] Defend the strategy selected for each source type.
