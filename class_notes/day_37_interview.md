# Day 37 Interview Guide — Incremental Loading and Upserts

**Project:** TechStore Incremental Data Loader  
**Audience:** Fresher / Junior Data Engineer  
**Important:** This lesson uses PostgreSQL syntax for `ON CONFLICT`. MySQL uses different upsert syntax. Always identify the database dialect in an interview.

---

## Part 1 — Core concepts with complete answers

### 1. What is a full load?

**Model answer:** A full load reads and loads the complete source dataset into the target for a defined scope. It is simple and useful for small datasets, initial loads or complete rebuilds. For very large datasets, repeating a full load can waste time, compute and network resources. A full load may also require careful replacement or staging so consumers do not see an incomplete target.

### 2. What is incremental loading?

**Model answer:** Incremental loading transfers only the source changes that occurred since a previously completed point. Changes can include inserts, updates and deletes. It reduces repeated work, but it introduces state-management challenges: the pipeline must identify changes, track progress, handle late updates and recover safely.

### 3. Does incremental loading mean only new rows?

**Model answer:** No. A new-row-only process handles inserts but can miss updates and deletes. A complete change strategy must define how each change type is detected. Timestamp columns can detect inserts and updates when maintained correctly; deletes may require soft-delete flags, change logs, CDC or periodic reconciliation.

### 4. What is a watermark?

**Model answer:** A watermark is a progress boundary used to identify which source records should be considered in the next extraction. It may be a monotonically increasing ID, an update timestamp or a compound value such as (updated_at, order_id). The extraction query uses the previous watermark and a new upper boundary to select a bounded change interval.

### 5. What is a checkpoint?

**Model answer:** A checkpoint is durable state recording the last safely completed progress point. It allows a pipeline to resume after failure. A checkpoint must not move ahead of data that has not been committed and verified in the destination. A watermark is the logical extraction boundary; a checkpoint is the persisted progress state, although teams sometimes use the terms together.

### 6. Why is a compound watermark useful?

**Model answer:** Timestamps are not always unique. Many rows may share the same updated_at value. If the query uses only `updated_at > last_timestamp`, records sharing the boundary timestamp can be missed. A compound watermark such as (updated_at, primary_key) provides a deterministic tie-breaker. The query compares timestamp first and key second.

### 7. What is an upsert?

**Model answer:** An upsert inserts a row when its key does not exist and updates the existing row when the key already exists. It is useful for applying inserts and updates to a target table. It requires a stable unique key and a clearly defined update policy. It does not automatically solve delete handling or guarantee correctness if the source change ordering is wrong.

### 8. What is PostgreSQL ON CONFLICT?

**Model answer:** PostgreSQL's `INSERT ... ON CONFLICT` handles a conflict with a unique or exclusion constraint. `DO NOTHING` ignores the conflicting insert; `DO UPDATE` updates the existing row. The conflict target identifies the key or constraint. I use it with a stable business key and test both insert and update behavior.

### 9. What does EXCLUDED mean in PostgreSQL?

**Model answer:** In an `ON CONFLICT DO UPDATE` clause, `EXCLUDED` refers to the row proposed for insertion that caused the conflict. For example, `amount = EXCLUDED.amount` assigns the incoming amount to the existing target row. It is not a general table alias outside that clause.

### 10. Why is a unique key important for upsert?

**Model answer:** The unique key defines which target record represents the same entity or event. Without a reliable key, the pipeline cannot know whether an incoming record is new or an update. A wrong key can overwrite unrelated records or create duplicates. The key must reflect the business grain of the target.

### 11. What is idempotency?

**Model answer:** An operation is idempotent when repeating it with the same input produces the same final state as applying it once. A deterministic upsert keyed by a stable identifier can be idempotent when updates are deterministic and source ordering is controlled. Appending the same batch repeatedly is not idempotent because it creates duplicates.

### 12. What is a lookback window?

**Model answer:** A lookback window re-reads a recent period before the saved watermark to catch late-arriving or delayed updates. For example, a pipeline may re-read the last two days and upsert by key. It improves resilience to late changes but increases repeated work and requires idempotent target writes.

### 13. How do you handle late-arriving updates?

**Model answer:** I define the source's update timestamp semantics, use a lookback window or change log, and compare source version or updated_at before overwriting target values. I ensure a late older record cannot replace a newer target version. I measure how late changes can arrive and set the lookback or reconciliation strategy accordingly.

### 14. How do you handle deletes?

**Model answer:** A timestamp query cannot detect a physically deleted row if no tombstone or change record remains. Options include soft-delete flags, CDC delete events, source audit tables or periodic full reconciliation. The chosen strategy must preserve deletion meaning and be agreed with downstream consumers.

### 15. What is CDC?

**Model answer:** Change Data Capture identifies inserts, updates and deletes from a source system, often by reading database logs or a change table. CDC can provide more complete change information than polling an update timestamp, but it requires infrastructure, ordering, retention and recovery management.

### 16. Why stage changes before applying them?

**Model answer:** Staging separates extraction and validation from target modification. I can inspect the batch, check schema, deduplicate by the agreed rule, validate counts and then apply it in a transaction. If the batch is invalid, I can stop before changing the trusted target.

### 17. When should the checkpoint advance?

**Model answer:** Only after the destination changes have been committed and verified according to the pipeline's correctness requirements. If the checkpoint advances first and the target write fails, the next run may skip missing data. If the target commits but checkpoint saving fails, replay must be safe through idempotent writes.

### 18. What happens if the target commit succeeds but checkpoint update fails?

**Model answer:** The next run may read the same interval again. If the target write is idempotent, the replay should not create duplicates or corrupt values. The pipeline then verifies the target and advances the checkpoint. This is safer than advancing the checkpoint before the target commit, which could lose data.

### 19. What happens if the checkpoint is saved before target commit?

**Model answer:** This creates a data-loss risk. If the checkpoint moves forward and the target transaction later rolls back or fails, the next run starts after data that was never loaded. The safe ordering is to commit and verify target data first, then persist the checkpoint, with idempotent replay handling uncertainty.

### 20. What is MERGE?

**Model answer:** MERGE is a SQL statement supported by some database systems that matches source rows to target rows and performs actions such as update, insert or delete based on match conditions. Syntax and capabilities vary by database. I explain the database dialect and verify supported behavior rather than assuming MERGE is identical everywhere.

---

## Part 2 — Project-based questions

### 21. Explain the TechStore Incremental Data Loader.

**Model answer:** The project starts with an initial source dataset and loads it into a target keyed by order_id. It then introduces a new order and an update to an existing order. The target is updated using an upsert, and the same change batch is replayed to demonstrate idempotency. The workbook then introduces timestamp watermark concepts and a checkpoint table. The central safety rule is that the checkpoint advances only after target data has been committed and verified.

### 22. How do you prove the upsert updated instead of duplicating?

**Model answer:** I inspect the target row count and query the specific key. After the first load, the key exists once. After an update, the row count remains the same while the expected column changes. After replaying the same batch, the count and final values remain unchanged. I also verify that the target has a unique constraint on the key.

### 23. Why does the project use PostgreSQL-specific syntax?

**Model answer:** The exercise uses PostgreSQL's `ON CONFLICT` so students can learn a concrete implementation. MySQL uses `INSERT ... ON DUPLICATE KEY UPDATE`, and other systems may use MERGE. The concept is portable, but the SQL syntax and edge cases are not identical.

### 24. What would you add before production?

**Model answer:** I would add a source change contract, compound watermark where needed, bounded extraction windows, staging, schema validation, unique constraints, transaction boundaries, idempotent upserts, delete handling, checkpoint audit history, late-arrival policy, reconciliation and operational metrics. I would test crash points between target commit and checkpoint save.

---

## Part 3 — Scenario questions

### 25. Two hundred rows have the same updated_at timestamp. How do you avoid missing some?

**Strong answer:** I use a deterministic compound boundary such as (updated_at, primary_key), or use an inclusive lookback with deduplication and idempotent upsert. I test the exact boundary case. I do not rely on a timestamp-only strict greater-than filter when timestamps can tie.

### 26. The pipeline runs twice for the same interval.

**Strong answer:** I expect a safe replay. I use a stable unique key, deterministic transformations and an idempotent upsert or deduplication mechanism. I verify that target counts and values remain correct after the second run. If the pipeline appends rows, I need a deduplication ledger or another mechanism before replay is safe.

### 27. A source record is updated with an older timestamp after a newer version is already in the target.

**Strong answer:** I compare source version or update timestamp and apply an update only when it is newer according to the source's ordering contract. I do not blindly overwrite the target with whichever row arrived last. If timestamps are unreliable, I need a source sequence/version or conflict-resolution policy.

### 28. The target write succeeds, but the process crashes before checkpoint save.

**Strong answer:** I assume the interval may be replayed. The target operation must be idempotent. On restart I re-read the interval, upsert the same keys, verify the destination and then save the checkpoint. I include this crash point in recovery testing.

### 29. The source physically deletes an order. Why does the timestamp watermark not find it?

**Strong answer:** The row no longer exists to be selected by the update query. I need a delete signal such as a soft-delete flag, CDC tombstone, audit log or periodic source-target reconciliation. The delete policy must be explicit so downstream systems know whether to remove, deactivate or retain historical records.

### 30. The source sends an update with a null amount. Should the upsert replace the target amount with null?

**Strong answer:** I check the source contract. Null may mean “clear the amount” or “value not supplied”. Those meanings are different. I should not use a blanket coalesce rule without business approval. The update semantics must define whether null overwrites, preserves the previous value or causes rejection.

---

## Part 4 — Practical revision checklist

- Explain full versus incremental loading.
- Distinguish inserts, updates and deletes.
- Explain watermark, checkpoint and compound watermark.
- Demonstrate PostgreSQL upsert.
- Prove idempotency by replaying the same batch.
- Explain late-arriving changes and lookback windows.
- Explain why checkpoint must not advance before target commit.
- Describe a delete strategy.
- Identify PostgreSQL/MySQL syntax differences.

## Part 5 — Self-assessment

| Skill | Score (0–2) |
|---|---:|
| Full and incremental loads | /2 |
| Watermark and checkpoint | /2 |
| Compound watermark | /2 |
| Upsert and unique key | /2 |
| Idempotency and replay | /2 |
| Late-arriving updates | /2 |
| Delete handling | /2 |
| Recovery ordering | /2 |

**13–16:** ready for junior-level discussion. **9–12:** repeat the replay and failure drills. **0–8:** revisit the class notes and rebuild the workbook from the initial load.
