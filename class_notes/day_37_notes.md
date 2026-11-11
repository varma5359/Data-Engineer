# Day 37: Incremental Loading and Upserts
**Phase:** 4 — Data Processing and Pipelines  
**Learning time:** 2–3 hours  
**Audience:** Complete beginners and freshers  
**Topic boundary:** This is the class lesson only. The separate project workbook and interview guide are not included.

> A daily pipeline should not always reload the entire source. Incremental loading identifies what is new or changed, transfers only that data and updates the destination safely. The difficult part is not just filtering rows. It is tracking progress without missing updates or creating duplicates.

## Learning outcomes

By the end, you should be able to:
- explain full load and incremental load;
- understand inserts, updates and deletes;
- explain watermark, high-water mark and checkpoint;
- choose between ID and timestamp watermarks;
- understand timestamp ties and late-arriving changes;
- build a bounded incremental extraction query;
- explain upsert and conflict handling;
- understand idempotency and safe replay;
- update existing records without creating duplicates;
- handle deleted source records through a documented strategy;
- save a checkpoint only after verified output;
- reconcile source changes and target state.

## Today's flow

`Read saved checkpoint -> Define extraction boundary -> Extract changes -> Validate -> Upsert destination -> Verify -> Save new checkpoint`

---

# 1. Why incremental loading exists

Imagine TechStore has 50 million orders.

A daily job runs at 02:00 and needs to update the analytics database.

A full load would read all 50 million rows every day, even if only 20,000 orders were added or changed.

That can create:
- unnecessary source database load;
- unnecessary network traffic;
- longer processing time;
- higher compute and storage costs;
- larger failure and recovery windows.

Incremental loading aims to process only new or changed records since a known point.

## 1.1 What is a full load?

A full load reads the entire defined source dataset and writes it to the destination.

Example:
- read all customers;
- replace the target customer snapshot;
- validate the full result.

Full loads are useful for:
- small reference tables;
- initial destination creation;
- controlled rebuilds;
- periodic reconciliation.

## 1.2 What is an incremental load?

An incremental load reads only the changes that occurred after the previous successful checkpoint.

Example:
- last successful order ID = 1000;
- new records have IDs 1001 to 1050;
- read only those records.

This simple strategy works only under specific source assumptions.

## 1.3 Incremental does not always mean new rows only

A record can be:
- inserted;
- updated;
- deleted.

A pipeline that reads only new IDs may miss updates to existing records and may never notice deletions.

Always define what types of source changes the pipeline must capture.

---

# 2. Understand source changes

## 2.1 Insert

A new business record is added.

Example:
- order_id 1051 is created.

## 2.2 Update

An existing business record changes.

Example:
- order_id 1051 changes from PENDING to PAID.

## 2.3 Delete

A source record is removed or marked deleted.

Example:
- a test customer is removed from an operational table.

A downstream analytical system may need to know that the record was deleted.

## 2.4 Why the pipeline must know the change type

Different changes require different destination behavior:
- insert -> create new destination row;
- update -> modify existing destination row;
- delete -> remove, mark deleted or retain history according to policy.

---

# 3. Understand a watermark

A watermark is a saved progress boundary used to determine what data should be processed next.

Examples:
- highest processed order ID;
- latest processed updated_at timestamp;
- last processed event sequence;
- source change-log position.

The watermark answers:

**“Up to what point has this pipeline safely processed the source?”**

## 3.1 High-water mark

A high-water mark is the greatest source position included in a completed extraction.

Example:
- current maximum source order_id = 1200;
- last saved watermark = 1000;
- process the range 1001 through 1200;
- after verification, save 1200.

## 3.2 Checkpoint

A checkpoint is the persisted record of processing progress.

A checkpoint may contain:
- source name;
- pipeline name;
- last processed ID;
- last processed timestamp;
- composite cursor;
- run ID;
- completion time;
- status.

A checkpoint must survive process restarts. A Python variable alone is not a durable checkpoint.

## 3.3 Watermark and checkpoint relationship

The watermark is the boundary value. The checkpoint stores that boundary and associated processing state.

Teams sometimes use these words interchangeably, but understand the distinction.

---

# 4. Build a simple ID-watermark example

## 4.1 Source data

| order_id | status | amount |
|---:|---|---:|
| 101 | PAID | 100 |
| 102 | SHIPPED | 200 |
| 103 | PENDING | 150 |
| 104 | PAID | 300 |

Suppose the saved watermark is 102.

The next extraction should read IDs greater than 102.

## 4.2 SQL query

```sql
SELECT order_id, status, amount
FROM orders
WHERE order_id > 102
ORDER BY order_id;
```

Expected result:

| order_id | status | amount |
|---:|---|---:|
| 103 | PENDING | 150 |
| 104 | PAID | 300 |

## 4.3 Why this is incremental

The query does not read IDs 101 and 102. It reads records beyond the saved boundary.

## 4.4 ID watermark assumptions

This approach assumes:
- IDs increase as new records are inserted;
- IDs are not reused;
- old records do not need to be re-read after updates;
- late-arriving records do not receive old IDs;
- the source's ID assignment is reliable.

If these assumptions are false, an ID-only watermark is incomplete.

---

# 5. Capture an upper boundary

A run should often define both a lower and upper boundary.

Suppose:
- saved watermark = 102;
- maximum ID observed at run start = 110.

Use:

```sql
SELECT order_id, status, amount
FROM orders
WHERE order_id > 102
  AND order_id <= 110
ORDER BY order_id;
```

The extraction scope is bounded.

Why is this useful?
- New records inserted during processing can be left for the next run.
- The current run has a clear expected range.
- Reconciliation can refer to a fixed upper boundary.
- The checkpoint should advance only to the verified upper boundary actually processed.

Do not save 110 if the query or output failed before processing all rows through 110.

---

# 6. Understand the weakness of ID watermarks

## 6.1 Existing record update

Suppose order_id 90 was loaded last week.

Today its status changes from PENDING to PAID.

The current watermark is 110.

Query:

```sql
SELECT *
FROM orders
WHERE order_id > 110;
```

Order 90 is not returned.

The pipeline misses the update.

## 6.2 When ID watermark is appropriate

It may be suitable for:
- append-only event tables;
- immutable transaction records;
- sources that guarantee increasing IDs and no later mutation.

## 6.3 When ID watermark is not enough

It is not enough when:
- records can be updated;
- records can be deleted;
- old IDs can arrive late;
- the source has backfilled records;
- IDs are assigned independently of commit order.

Choose a change-capture strategy that matches the source.

---

# 7. Timestamp watermark

A source table may have an `updated_at` column.

It changes when a record is inserted or updated.

Example:

```sql
SELECT order_id, status, amount, updated_at
FROM orders
WHERE updated_at > TIMESTAMP '2026-11-10 00:00:00'
ORDER BY updated_at, order_id;
```

This selects rows updated after the saved timestamp.

## 7.1 Why updated_at is useful

Unlike an ID-only watermark, an update timestamp can identify changes to older records when the source maintains it correctly.

## 7.2 Source contract questions

Ask:
- Is updated_at set on insert?
- Is it changed on every update?
- Is it generated by the database or application?
- Is it stored in UTC?
- What is its precision?
- Can two records share the same timestamp?
- Can an update arrive late?
- Can the timestamp be backdated?

A timestamp field is only useful if its semantics are reliable.

---

# 8. Understand timestamp ties

Two records can have the same updated_at value.

Example:

| order_id | updated_at |
|---:|---|
| 201 | 2026-11-10 10:00:00 |
| 202 | 2026-11-10 10:00:00 |
| 203 | 2026-11-10 10:01:00 |

Suppose a run processes order 201 and saves watermark 10:00:00.

The next query uses:

```sql
WHERE updated_at > TIMESTAMP '2026-11-10 10:00:00'
```

Order 202 is excluded because its timestamp equals the watermark.

This is a data-loss risk.

## 8.1 Composite watermark

Use timestamp plus a unique tie-breaker such as order_id.

The cursor becomes:
- last_updated_at;
- last_order_id.

## 8.2 Composite comparison

```sql
SELECT order_id, status, amount, updated_at
FROM orders
WHERE
    updated_at > %s
    OR (updated_at = %s AND order_id > %s)
ORDER BY updated_at, order_id;
```

Parameters:
- first timestamp = saved timestamp;
- second timestamp = same saved timestamp;
- third value = saved order ID.

This compares the pair in a deterministic order.

The exact comparison must match the source's ordering and uniqueness guarantees.

## 8.3 Why order by both fields?

The ORDER BY matches the composite cursor order:
1. updated_at;
2. order_id.

If the query orders differently, checkpoint logic can become inconsistent.

---

# 9. Late-arriving updates

A late-arriving update is a change that becomes visible after the pipeline has already passed the relevant business time.

Example:
- order was created on Monday;
- partner sends a correction on Wednesday;
- business date remains Monday;
- updated_at is Wednesday.

If the pipeline filters only by order_date, it may never see the correction.

Use the correct change timestamp or source change feed.

## 9.1 Event time versus update time

- event time: when the business event occurred;
- created time: when the source record was first created;
- updated time: when the source record last changed;
- ingestion time: when the pipeline received it.

These fields answer different questions.

## 9.2 Overlap window

One strategy is to re-read a small time window before the saved watermark.

Example:
- saved watermark: 12:00;
- next extraction starts at 11:55;
- some records are read again;
- destination upsert handles the replay.

This can help with late visibility and timestamp precision, but it requires a stable key and idempotent destination behavior.

## 9.3 Overlap trade-off

A larger overlap:
- reduces risk of missing late changes;
- increases repeated reads;
- requires stronger deduplication.

A smaller overlap:
- reduces repeated work;
- may miss late changes.

Choose based on measured source delay and business requirements.

---

# 10. Understand upsert

Upsert combines insert and update behavior.

- If the key does not exist, insert a new row.
- If the key already exists, update the existing row according to the rule.

Upsert is useful when incremental extracts can contain both new and changed records.

## 10.1 PostgreSQL example

```sql
INSERT INTO target_orders (order_id, status, amount)
VALUES (101, 'PAID', 150.00)
ON CONFLICT (order_id)
DO UPDATE SET
    status = EXCLUDED.status,
    amount = EXCLUDED.amount;
```

Explanation:
- INSERT INTO identifies the target table.
- The column list defines the fields.
- VALUES provides the incoming record.
- ON CONFLICT identifies a uniqueness conflict.
- order_id is the conflict key.
- EXCLUDED refers to the proposed incoming row.
- DO UPDATE changes the existing target record.

This is PostgreSQL syntax. MySQL and SQL Server use different upsert approaches.

## 10.2 Why a unique key is required

The database needs a reliable way to determine whether the incoming record already exists.

The conflict key should represent the business identity.

Do not use a non-unique description or customer name as the key.

## 10.3 Upsert does not mean blindly overwrite everything

The business may need:
- latest version wins;
- only newer updated_at values replace older values;
- selected columns update;
- historical versions are retained;
- invalid transitions are rejected.

Upsert syntax performs a database operation. It does not decide the business policy.

## 10.4 Avoid overwriting newer target data with older source data

Suppose the target already has updated_at 12:00 and a replayed source row has updated_at 11:00.

A simple upsert may overwrite the newer target.

Use a conditional update or version check so older data cannot replace newer data.

---

# 11. Idempotency and safe replay

## 11.1 What is idempotency?

An operation is idempotent when repeating it produces the same intended final state as performing it once.

Example:
- upsert order 101 with status PAID;
- run the same upsert again;
- target still contains one order 101 with status PAID.

## 11.2 Why replay happens

- network failure;
- process crash;
- scheduler retry;
- operator rerun;
- destination committed but checkpoint was not saved.

Pipelines should expect replay.

## 11.3 Insert-only behavior can create duplicates

If the target has no unique key and the pipeline always inserts, rerunning the same batch can create duplicate rows.

A stable key and destination constraint are important.

## 11.4 Idempotent upsert

A stable primary/unique key plus deterministic update rules can make replay safe.

But idempotency must include all side effects. Sending a notification or charging a card is not automatically safe just because a database row uses upsert.

---

# 12. Build a small Python upsert simulation

This section demonstrates destination behavior without a database connection.

## 12.1 Existing target records

```python
target_orders = {
    101: {"order_id": 101, "status": "PENDING", "amount": 100},
    102: {"order_id": 102, "status": "PAID", "amount": 200}
}
```

The dictionary key is order_id. It represents one target row per order.

## 12.2 Incoming changes

```python
incoming_orders = [
    {"order_id": 102, "status": "SHIPPED", "amount": 220},
    {"order_id": 103, "status": "PAID", "amount": 150}
]
```

Order 102 already exists and has changed. Order 103 is new.

## 12.3 Apply upsert behavior

```python
for order in incoming_orders:
    order_id = order["order_id"]
    target_orders[order_id] = order

print(target_orders)
```

Expected final state:
- order 101 remains;
- order 102 is updated;
- order 103 is inserted.

The dictionary assignment replaces the value for an existing key and creates a new key when absent.

## 12.4 Replay the same batch

```python
for order in incoming_orders:
    order_id = order["order_id"]
    target_orders[order_id] = order

print(len(target_orders))
```

Expected output:

```text
3
```

The target still has three keys.

This simulation illustrates stable-key replacement. It does not model database transactions, concurrent writers or durable checkpoints.

---

# 13. Handle deletes

An incremental pipeline that captures only inserts and updates may leave deleted records in the destination.

Possible delete strategies:
- hard delete the target row;
- soft delete using is_deleted;
- store a tombstone event;
- use database change data capture;
- periodically reconcile source and target.

## 13.1 What is a tombstone?

A tombstone is a record or marker indicating that a business entity was deleted.

Example:

```json
{
  "order_id": 101,
  "operation": "DELETE",
  "changed_at": "2026-11-11T10:00:00Z"
}
```

The downstream pipeline can use this event to apply the deletion policy.

## 13.2 Why not infer deletes from absence in one incremental batch?

A row missing from today's changes does not mean it was deleted. It may simply have remained unchanged.

To infer deletions from absence, compare complete source and target snapshots or use an explicit change feed.

---

# 14. Save the checkpoint in the correct order

The checkpoint is a promise that all data up to a boundary has been safely processed.

Unsafe sequence:

`Save checkpoint -> Write output -> Failure`

If output fails after the checkpoint is saved, the next run may skip missing data.

Safer conceptual sequence:

`Extract -> Validate -> Write/upsert -> Verify -> Commit/publish -> Save checkpoint`

The exact transaction design depends on the destination and checkpoint store.

## 14.1 What if output succeeds but checkpoint fails?

The next run may replay the same source range.

This is why idempotent writes are important.

## 14.2 What if checkpoint succeeds but output is incomplete?

This is dangerous because data can be permanently skipped.

Do not advance progress until the target result is durable and verified.

## 14.3 Keep checkpoint metadata

Useful fields:
- pipeline name;
- source system;
- watermark value;
- run ID;
- started_at;
- completed_at;
- input count;
- written count;
- status;
- source range;
- destination reference.

---

# 15. Reconcile an incremental load

## 15.1 Compare extracted and written counts

Record:
- rows extracted;
- rows accepted;
- rows rejected;
- rows inserted;
- rows updated;
- rows unchanged;
- rows failed.

These are different metrics. Do not treat them as interchangeable.

## 15.2 Compare unique keys

Verify that the target has one row per business key when the target model requires uniqueness.

## 15.3 Compare versions

If the source has updated_at or version number, ensure the target does not contain an older version than the source for the processed scope.

## 15.4 Compare important values

Check:
- status distribution;
- amount totals;
- minimum and maximum timestamps;
- null counts;
- deleted record handling.

## 15.5 Verify the checkpoint boundary

The saved checkpoint should not exceed the maximum source position that was successfully written and verified.

---

# 16. Common incremental-loading mistakes

## Mistake 1 — Use ID watermark for mutable records

**Risk:** Updates to old IDs are missed.

**Correction:** Use updated_at, CDC or another source change mechanism.

## Mistake 2 — Use timestamp greater-than without tie handling

**Risk:** Rows sharing the boundary timestamp may be skipped.

**Correction:** Use a composite cursor or overlap strategy.

## Mistake 3 — Save checkpoint before output commit

**Risk:** Data can be skipped permanently.

**Correction:** Save progress after durable output and verification.

## Mistake 4 — Insert every replayed row

**Risk:** Duplicate destination records.

**Correction:** Use a stable key and idempotent destination behavior.

## Mistake 5 — Ignore deletes

**Risk:** Stale rows remain in the target.

**Correction:** Capture tombstones, CDC or perform controlled reconciliation.

## Mistake 6 — Overwrite a newer row with an older replay

**Risk:** Target regresses to stale data.

**Correction:** Compare source version or updated timestamp before updating.

## Mistake 7 — Assume a timestamp is trustworthy

**Risk:** Clock differences, precision and late updates create gaps.

**Correction:** Understand timestamp ownership, timezone and source guarantees.

---

# 17. Practical exercises

## Exercise 1 — ID watermark

Saved watermark is 500. Source contains IDs 501, 502 and 503.

Expected extraction: IDs greater than 500.

## Exercise 2 — Update to old ID

Saved watermark is 500. Record 100 changes status.

Will `WHERE id > 500` return it?

Expected answer: no.

## Exercise 3 — Timestamp tie

Two records have timestamp 10:00:00. The saved watermark is 10:00:00.

Why can `updated_at > watermark` miss one?

Expected answer: equal timestamps are excluded.

## Exercise 4 — Upsert

Target has order 10. Incoming order 10 has a new status.

Expected behavior: update the existing row according to the approved version rule.

## Exercise 5 — Replay

The same incoming batch is processed twice.

What prevents duplicate target rows?

Expected answer: stable unique key and idempotent upsert behavior.

## Exercise 6 — Delete

A source record disappears from one incremental result.

Can we assume it was deleted?

Expected answer: no. It may simply be unchanged. Use an explicit deletion signal or complete reconciliation.

## Exercise 7 — Checkpoint order

Should the watermark be saved before writing the output?

Expected answer: no. Save only after durable output and verification.

## Exercise 8 — Older replay

Target updated_at is 14:00. Replayed source row updated_at is 13:00.

Should it overwrite the target?

Expected answer: no, not when the business rule says newer source versions win.

---

# 18. Debugging checklist

When an incremental pipeline misses or duplicates data:

1. Inspect the saved checkpoint.
2. Confirm the lower boundary.
3. Confirm the upper boundary.
4. Check the source's change semantics.
5. Confirm ID ordering assumptions.
6. Check updated_at precision and timezone.
7. Look for timestamp ties.
8. Look for late-arriving updates.
9. Check whether deletes are represented.
10. Verify source query ordering.
11. Check destination unique constraints.
12. Check upsert update conditions.
13. Confirm replay does not create duplicates.
14. Compare source and destination versions.
15. Verify output before advancing the checkpoint.
16. Inspect run metadata and failure point.

---

# 19. Day 37 revision checklist

You should be able to explain:
- full load;
- incremental load;
- insert, update and delete;
- watermark and checkpoint;
- ID watermark assumptions;
- bounded high-water mark;
- updated_at watermark;
- timestamp ties;
- composite cursor;
- late-arriving updates;
- overlap window;
- upsert;
- PostgreSQL ON CONFLICT;
- EXCLUDED row;
- stable unique key;
- idempotency;
- replay;
- tombstone;
- safe checkpoint ordering;
- count, key and version reconciliation.

---

# 20. Key takeaways

1. Incremental loading avoids repeatedly processing unchanged data.
2. ID watermarks are suitable only when source assumptions support them.
3. Updates to older IDs require another change-capture strategy.
4. Timestamp watermarks need tie and late-arrival handling.
5. A composite cursor can make ordering deterministic.
6. Upsert inserts new rows and updates existing rows according to a rule.
7. Stable keys make replay safer.
8. Idempotency must cover the intended side effects.
9. Deletes need explicit handling.
10. Checkpoints must be saved after durable, verified output.
11. Reconcile counts, keys and versions before declaring success.

## What comes next

Day 38 focuses on pipeline reliability and failure handling. You will learn how to design retries, handle partial writes, recover from crashes, make replays safe and test failure scenarios deliberately.

**End of Day 37 class notes.**

---

# 21. Change Data Capture and other change sources

An updated_at column is not the only way to identify changes.

**Change Data Capture (CDC)** captures database changes such as inserts, updates and deletes. A CDC event may contain an operation type, primary key, changed values, transaction position and event time.

Some tools read database transaction logs and convert committed changes into events. A pipeline must understand event ordering, duplicate delivery, delete events, schema changes and source log retention.

## 21.1 Snapshot plus CDC

A common pattern is:
1. take a consistent initial snapshot;
2. record the matching source log position;
3. load the snapshot;
4. consume changes after that position;
5. apply changes in order;
6. persist the new source position.

The snapshot and change position must be coordinated. If the boundary is wrong, changes can be missed or applied twice.

## 21.2 CDC still needs idempotency

Events may be delivered again after restart or replay. Use a stable event identity or source position and apply changes safely.

# 22. Design a checkpoint table

A durable checkpoint is commonly stored in a database or another reliable state store.

```sql
CREATE TABLE IF NOT EXISTS pipeline_checkpoint (
    pipeline_name VARCHAR(100) PRIMARY KEY,
    last_order_id BIGINT,
    last_updated_at TIMESTAMP,
    last_run_id VARCHAR(100),
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

This is an illustrative PostgreSQL table. A real design may store a composite cursor, source log position or separate run history.

## 22.1 Read a checkpoint

```sql
SELECT pipeline_name, last_order_id, last_updated_at
FROM pipeline_checkpoint
WHERE pipeline_name = 'daily_orders';
```

The pipeline reads its last saved progress before defining the next extraction range.

## 22.2 One checkpoint per source scope

If a pipeline reads multiple independent sources or partitions, it may need one checkpoint per source/partition. A single global value can incorrectly mix independent progress boundaries.

## 22.3 Do not save an unverified maximum

The maximum source value observed is not necessarily the maximum value safely written. Advance only to a boundary that the run fully processed and verified.

# 23. Conditional upsert protects newer target data

A simple upsert can overwrite a newer target row with an older replay. PostgreSQL can condition the conflict update.

```sql
INSERT INTO target_orders (order_id, status, amount, updated_at)
VALUES (%s, %s, %s, %s)
ON CONFLICT (order_id)
DO UPDATE SET
    status = EXCLUDED.status,
    amount = EXCLUDED.amount,
    updated_at = EXCLUDED.updated_at
WHERE target_orders.updated_at < EXCLUDED.updated_at;
```

Explanation:
- incoming values are proposed for insertion;
- order_id identifies a conflict;
- the existing target row is updated only when its timestamp is older;
- an older replay does not replace a newer target version.

This assumes updated_at is trustworthy and comparable. If timestamps can tie or be unreliable, use a source version or sequence.

## 23.1 Equal timestamps

If two versions have equal updated_at values, the condition above does not update the target. That may be correct or may require a secondary version field. Define the rule first.

## 23.2 Version number

A monotonically increasing source version can be a stronger ordering signal than wall-clock time when the source provides it reliably.

# 24. Test incremental boundaries explicitly

Create test cases for:
- first record after the watermark;
- record exactly equal to the watermark;
- two records with the same timestamp;
- record inserted during extraction;
- old record updated after the watermark;
- late-arriving record;
- duplicate event;
- deleted record;
- empty change range.

For every case, write down the source state before the run, saved checkpoint, expected extracted records, expected destination state and expected new checkpoint.

A boundary test is more useful than testing only a normal happy-path example.

# 25. Incremental-load design decision guide

| Source behavior | Possible strategy |
|---|---|
| Append-only, increasing unique ID | ID watermark |
| Existing rows can change and updated_at is reliable | Timestamp/composite watermark |
| Timestamp ties are common | Composite cursor or overlap with deduplication |
| Deletes must be captured | CDC/tombstone or full reconciliation |
| Source has transaction-log change feed | CDC position |
| Source timestamps are unreliable | Source-provided sequence/change token |
| Small reference table | Periodic full refresh may be simpler |

There is no single incremental strategy that is correct for every source.

## 25.1 Document assumptions

Before deployment, document source key, change mechanism, timestamp timezone and precision, tie handling, late-arrival policy, delete handling, overlap duration, checkpoint store, destination idempotency rule and reconciliation process.

# 26. Additional practice: decide which cursor to use

**Scenario A:** An append-only event table uses an increasing event_id and old events never change. An ID watermark may be suitable.

**Scenario B:** An order table updates status several times. An ID watermark alone is insufficient; use updated_at, CDC or another reliable change signal.

**Scenario C:** Two updates can share the same timestamp. Use a composite cursor or overlap/deduplication strategy.

**Scenario D:** The source physically deletes rows and the target must reflect deletion. A timestamp query over remaining rows cannot discover the missing row. Use CDC, tombstones or full reconciliation.

**Scenario E:** A small reference table has 500 rows. A full refresh may be simpler and safer than a complex incremental mechanism.

# 27. Review your incremental design

Before writing code, answer:
1. What is the business key?
2. Can existing rows change?
3. Can rows be deleted?
4. What field or log position identifies a change?
5. Can multiple changes share the same cursor value?
6. Can changes arrive late?
7. How is the upper boundary fixed?
8. Where is the checkpoint stored?
9. What makes destination writes idempotent?
10. How will source and target be reconciled?

If these answers are unknown, the design is not ready for production.

**End of expanded Day 37 teaching notes.**