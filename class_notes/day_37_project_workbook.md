
# Day 37 Project Workbook — TechStore Incremental Data Loader

**Goal:** Load only new or changed orders, upsert by a stable key, replay safely and advance a checkpoint only after destination verification.  
**Database dialect:** PostgreSQL syntax is used below; MySQL upsert syntax differs.

## 1. Business scenario

A full reload of millions of orders every day is wasteful. The target needs inserts, updates and a documented delete strategy. A watermark is the source progress boundary. A checkpoint is the persisted record of progress. Neither should advance before the target is verified.

## 2. Simulate the initial load

~~~python
source = [
    {"order_id": 101, "amount": 100, "updated_at": "2026-01-01 09:00:00"},
    {"order_id": 102, "amount": 200, "updated_at": "2026-01-01 10:00:00"},
    {"order_id": 103, "amount": 150, "updated_at": "2026-01-01 11:00:00"}
]
target = {}
for row in source:
    target[row["order_id"]] = row.copy()
print("Initial target:", len(target))
print(target[102])
~~~

Expected: 3 target rows; order 102 amount is 200. The dictionary represents a destination keyed by order_id.

## 3. Simulate a new order and an update

~~~python
changes = [
    {"order_id": 102, "amount": 225, "updated_at": "2026-01-02 08:00:00"},
    {"order_id": 104, "amount": 90, "updated_at": "2026-01-02 08:30:00"}
]
for row in changes:
    target[row["order_id"]] = row.copy()
print("Target rows:", len(target))
print("Order 102:", target[102])
print("Order 104:", target[104])
~~~

Expected: 4 target rows; order 102 is updated to 225; order 104 is inserted with 90. This is upsert behavior.

## 4. Replay the same changes

~~~python
for row in changes:
    target[row["order_id"]] = row.copy()
print("Rows after replay:", len(target))
print("Order 102 amount:", target[102]["amount"])
~~~

Expected: still 4 rows and amount 225. This in-memory example is idempotent for complete records and stable keys. Real systems must also handle event ordering and partial updates.

## 5. Understand timestamp watermarks

A mutable source record can be updated long after its ID was first created. Therefore an ID-only watermark can miss updates. A timestamp cursor can help:

~~~sql
SELECT order_id, amount, updated_at
FROM source_orders
WHERE updated_at > TIMESTAMP '2026-01-01 23:59:59'
ORDER BY updated_at, order_id;
~~~

This assumes updated_at is reliable. Timestamp ties need care: if two rows share the checkpoint timestamp, strict greater-than may miss one. Use a composite cursor such as (updated_at, order_id), or an overlap window plus deduplication.

## 6. Create a PostgreSQL target

~~~sql
CREATE TABLE IF NOT EXISTS target_orders (
    order_id INTEGER PRIMARY KEY,
    amount NUMERIC(12, 2) NOT NULL,
    updated_at TIMESTAMP NOT NULL
);
~~~

The primary key prevents two target rows for the same order_id.

## 7. PostgreSQL upsert

~~~sql
INSERT INTO target_orders (order_id, amount, updated_at)
VALUES
    (102, 225.00, TIMESTAMP '2026-01-02 08:00:00'),
    (104, 90.00, TIMESTAMP '2026-01-02 08:30:00')
ON CONFLICT (order_id)
DO UPDATE SET
    amount = EXCLUDED.amount,
    updated_at = EXCLUDED.updated_at
WHERE EXCLUDED.updated_at >= target_orders.updated_at;
~~~

ON CONFLICT handles an existing key. EXCLUDED refers to the incoming row. The WHERE condition prevents an older replay from replacing a newer target value. Define an explicit policy for equal timestamps.

Verify:

~~~sql
SELECT order_id, amount, updated_at
FROM target_orders
ORDER BY order_id;
~~~

Orders 102 and 104 must have the values above. Orders 101 and 103 remain from the initial load.

## 8. Checkpoint table and safe ordering

~~~sql
CREATE TABLE IF NOT EXISTS pipeline_checkpoint (
    pipeline_name TEXT PRIMARY KEY,
    last_updated_at TIMESTAMP NOT NULL,
    last_order_id INTEGER NOT NULL,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
~~~

Safe order:
1. Read old checkpoint.
2. Capture a bounded upper source boundary.
3. Extract changes within the boundary.
4. Validate and upsert target.
5. Verify target keys, counts and versions.
6. Commit target transaction.
7. Save the new checkpoint.

If target commit succeeds but checkpoint update fails, replay the same range. Idempotent upsert makes that safe. Never advance the checkpoint before durable output.

## 9. Delete strategy

Absence from one incremental batch does not prove deletion. Use a source delete flag, CDC delete event, tombstone or periodic full reconciliation. Decide whether to physically delete or mark the target row inactive.

## 10. Failure drills

- Replay the same changes twice; target remains four rows.
- Try an older timestamp for order 102; it must not replace the newer value.
- Create two rows with the same timestamp; explain the composite cursor.
- Simulate target success but checkpoint failure; replay the same interval.
- Force a target constraint violation; rollback and leave checkpoint unchanged.

## 11. Troubleshooting

| Symptom | Cause | Correction |
|---|---|---|
| Duplicate target key | No unique constraint or insert-only logic | Add primary key and upsert |
| Older value overwrites newer | No version comparison | Compare timestamp or source version |
| Tied timestamp row missed | Timestamp-only strict greater-than cursor | Composite cursor or overlap plus dedupe |
| Changes missed | ID watermark on mutable rows | Update cursor or CDC |
| Checkpoint advanced too early | Checkpoint saved before output commit | Commit and verify target first |
| Deletes missing | Source does not expose deletes | Tombstones, CDC or snapshot reconciliation |

## 12. Acceptance checklist

- [ ] Initial load understood.
- [ ] New rows inserted.
- [ ] Existing rows updated.
- [ ] Replay creates no duplicates.
- [ ] Older events cannot overwrite newer values.
- [ ] Timestamp ties considered.
- [ ] Delete strategy documented.
- [ ] Checkpoint advances only after verified target commit.

**Project explanation:** "I read source changes after a saved boundary and upsert by a stable order key. I protect newer target versions from old replays, verify the destination and only then advance the checkpoint."
