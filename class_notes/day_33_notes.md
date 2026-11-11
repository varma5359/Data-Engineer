# Day 33: Database Ingestion — Beginner Notebook Class

**Phase:** 4 — Data Processing and Pipelines  
**Time:** 2–3 hours  
**Project:** TechStore Database Extractor  
**Level:** Complete beginner / fresher  
**Tools:** Python, Psycopg 3, PostgreSQL, Jupyter Notebook  
**Prerequisites:** SQL basics, Python lists/loops, Day 30 extraction and Day 29 database connection

---

# 0. Today's business problem

A company has customer/order data in an operational database. Analytics needs a controlled copy, but the source database is used by live applications. We must extract data efficiently and safely.

Today we build a small extractor that:
1. connects with a read-only account;
2. selects explicit columns;
3. uses safe SQL parameters;
4. fetches data in batches;
5. writes a local extract;
6. checks counts, keys and boundaries;
7. explains incremental extraction limitations.

Flow:
`Source DB -> Safe SQL -> Cursor -> Batches -> Output -> Reconciliation -> Checkpoint`

---

# 1. Database extraction vocabulary

| Term | Beginner meaning |
|---|---|
| Source database | Database where operational data lives |
| Connection | Communication session with DB |
| Cursor | Object used to execute SQL/read results |
| Result set | Rows returned by query |
| Fetch | Transfer result rows into Python |
| Batch | A bounded group of rows |
| Read-only account | Can read but cannot change source data |
| Query timeout | Maximum allowed query duration |
| Watermark | Saved progress boundary |
| Snapshot | Consistent view of data at a point, depending on DB isolation |
| CDC | Change Data Capture; captures inserts/updates/deletes from change stream |
| Reconciliation | Evidence that extracted output matches expected source |

---

# 2. Why source safety matters

Operational database supports customer actions—checkout, booking, payments, inventory. An expensive extraction can compete for CPU, I/O, memory or locks.

Safer choices:
- read only access;
- explicit columns;
- source-side filters;
- indexed predicates where suitable;
- bounded batches;
- query timeout and cancellation;
- schedule/backfill coordination;
- approved replica if appropriate;
- monitor source load.

**Important:** A read-only account prevents writes but does not prevent an expensive SELECT. Permission and performance safety are different controls.

---

# 3. Confirm connection and source table

Use the TechStore DB from Day 19/20.

```python
import os
import psycopg

connection = psycopg.connect(
    host=os.environ["PGHOST"],
    port=os.environ["PGPORT"],
    dbname=os.environ["PGDATABASE"],
    user=os.environ["PGUSER"],
    password=os.environ["PGPASSWORD"]
)

cursor = connection.cursor()
cursor.execute("SELECT current_database(), current_user")
print(cursor.fetchone())
```

Expected shape:
```text
('techstore', 'your_database_user')
```
Actual user/database depend on your setup.

Check table:
```sql
SELECT COUNT(*) FROM orders;
```
The count is your actual seed/source size. Record it; do not invent a number.

---

# 4. Explicit query and parameterized values

## 4.1 Query selected fields

```python
cursor.execute("""
SELECT order_id, customer_id, created_at, status, total_amount
FROM orders
ORDER BY order_id
LIMIT 5
""")

sample_rows = cursor.fetchall()
print("Sample rows:", len(sample_rows))
for row in sample_rows:
    print(row)
```

Expected: up to five rows, ordered by order_id. Exact rows depend on TechStore seed data.

## 4.2 Parameterized filter

```python
customer_id = 501

cursor.execute("""
SELECT order_id, customer_id, status, total_amount
FROM orders
WHERE customer_id = %s
ORDER BY order_id
""", (customer_id,))

customer_orders = cursor.fetchall()
print("Orders for customer:", len(customer_orders))
```

The placeholder is not quoted. The value is supplied separately. For a single parameter, `(customer_id,)` is a tuple; the comma is essential.

**Never** build SQL by concatenating an untrusted customer ID or text. Parameters protect values and preserve type handling.

---

# 5. Fetch one, many and all

```python
cursor.execute("SELECT order_id FROM orders ORDER BY order_id")

one_row = cursor.fetchone()
print("One:", one_row)

next_rows = cursor.fetchmany(3)
print("Next batch:", next_rows)

remaining = cursor.fetchall()
print("Remaining:", len(remaining))
```

Each fetch consumes rows from the cursor. If you call fetchone, the next fetchmany begins after that row. If you need a fresh result, execute the query again.

**fetchall** is suitable for small data. Large datasets require bounded processing.

---

# 6. Bounded batch extraction

## 6.1 Understand batch size

Suppose 25 rows and batch size 10:
- Batch 1: 10
- Batch 2: 10
- Batch 3: 5

The final batch can be smaller. Do not discard it.

## 6.2 Execute and fetch in batches

```python
batch_size = 10

cursor.execute("""
SELECT order_id, customer_id, created_at, status, total_amount
FROM orders
ORDER BY order_id
""")

total_fetched = 0
batch_number = 0

while True:
    batch = cursor.fetchmany(batch_size)

    if not batch:
        break

    batch_number += 1
    total_fetched += len(batch)

    print(
        "Batch:", batch_number,
        "Rows:", len(batch),
        "Total fetched:", total_fetched
    )
```

The printed values depend on source row count. The loop stops when fetchmany returns an empty list.

## 6.3 Why this is better than fetchall

Batching limits the number of result rows held at once. But note: driver/server cursor behavior matters. For very large PostgreSQL results, use a named/server-side cursor or documented streaming approach; ordinary client-side cursor may buffer more than expected. Learn the driver semantics before claiming memory is fully bounded.

---

# 7. Write batches to CSV instead of collecting all rows

## 7.1 Stream to a file

```python
import csv

output_path = "../output/orders_full_extract.csv"

cursor.execute("""
SELECT order_id, customer_id, created_at, status, total_amount
FROM orders
ORDER BY order_id
""")

column_names = [column.name for column in cursor.description]

with open(output_path, "w", newline="", encoding="utf-8") as file:
    writer = csv.writer(file)
    writer.writerow(column_names)

    total_written = 0

    while True:
        batch = cursor.fetchmany(500)

        if not batch:
            break

        writer.writerows(batch)
        total_written += len(batch)

print("Rows written:", total_written)
```

This writes each batch as it is fetched instead of keeping every row in one Python list.

For large production extracts, consider server-side cursors, query timeout, temporary output, file validation and atomic publication.

---

# 8. Verify extraction

## 8.1 Compare source count and output count

```sql
SELECT COUNT(*) FROM orders;
```

Compare with Python's total_written.

If filters are used, source count must use the same WHERE condition.

## 8.2 Key checks

For a small dataset:
- inspect first/last order ID;
- verify IDs are unique;
- compare key count;
- compare amount totals if business rules permit;
- check nulls and date range.

For large data, choose efficient checks (database constraints, aggregates, key ranges, sampled/full reconciliation) rather than loading all IDs into memory.

---

# 9. Incremental extraction with ID watermark

## 9.1 Example for append-only records

Suppose last successfully extracted ID = 1000.

```python
last_id = 1000
cursor.execute("""
SELECT order_id, customer_id, created_at, status, total_amount
FROM orders
WHERE order_id > %s
ORDER BY order_id
""", (last_id,))
```

This extracts IDs greater than 1000.

**Limitation:** It only works for newly inserted higher IDs. If order 500 is updated or deleted, this query will not detect it.

## 9.2 Bound the run with upper ID

Capture a maximum ID at run start:

```sql
SELECT MAX(order_id) FROM orders;
```

Suppose upper_id=1500. Query:
```sql
SELECT order_id, customer_id, created_at, status, total_amount
FROM orders
WHERE order_id > 1000
  AND order_id <= 1500
ORDER BY order_id;
```

This bounds an append-only extraction window. It does not create a consistent snapshot across concurrent changes and does not capture updates/deletes.

---

# 10. Timestamp watermark and tie handling

If records have updated_at, a timestamp alone can have ties. Use timestamp plus unique key.

Concept:
`updated_at > last_time OR (updated_at = last_time AND order_id > last_id)`

Order by updated_at, order_id.

A safe implementation also needs:
- timezone and precision agreement;
- late-update strategy;
- overlap/replay;
- destination idempotency;
- checkpoint after durable write;
- delete capture strategy.

Do not claim timestamp watermark guarantees every change unless source semantics support it.

---

# 11. Source consistency and replica lag

A query may run while source records are changing. A plain SELECT does not automatically guarantee that multiple batches represent one consistent point in time; transaction isolation matters.

If using a read replica:
- replication lag may cause data to be behind primary;
- a boundary from primary can exceed replica's applied changes;
- monitor lag and choose safe overlap/position;
- reconcile later.

For exact change capture, use database-approved snapshot/CDC strategy.

---

# 12. Failure and recovery

| Failure | Safe response |
|---|---|
| Connection drops | Reconnect and replay from last durable batch |
| Query timeout | Investigate plan/source load; don't retry forever |
| Destination write fails | Do not advance checkpoint |
| File write partial | Temp file, validate, publish safely |
| Crash after write before checkpoint | Replay idempotently |
| Checkpoint before write | Potential data loss; correct ordering |
| Source schema changed | Stop, inspect contract, version mapping |
| Duplicate source key | Investigate join/key/change semantics |
| Replica behind | Wait/overlap/reconcile; don't advance unsafely |

---

# 13. Mini-project: TechStore Database Extractor

## Requirement
Extract all or a bounded subset of orders from PostgreSQL to CSV using batch fetching and verification.

## Milestones
1. Confirm database/table/schema.
2. Use read-only credentials.
3. Define explicit columns and filter.
4. Record source count/upper boundary.
5. Execute query and fetch batches.
6. Write temp CSV batch by batch.
7. Count output records and verify key/boundary.
8. Publish only after checks.
9. Save run summary and checkpoint.
10. Test interruption/replay and explain watermark limitation.

## Summary fields
`run_id, source, query_window, upper_boundary, rows_read, rows_written, batch_count, duration, status, checkpoint`

A successful run must prove that output is durable and reconciliation passes.

---

# 14. Exercises with answers

1. Why use read-only account?
2. Does read-only prevent expensive queries?
3. Why use explicit columns?
4. Why use parameter placeholders?
5. What does fetchmany return when no rows remain?
6. Why can fetchall exhaust memory?
7. Why can ordinary cursor still buffer results?
8. What can ID watermark miss?
9. Why use timestamp+ID?
10. Why checkpoint after destination commit?
11. What is replica lag?
12. What does reconciliation prove?

**Answer guide**
1. Prevent accidental source changes.
2. No; it still consumes resources.
3. Stable contract and less data.
4. Safe value binding and correct types.
5. Empty list.
6. All remaining rows materialize in memory.
7. Driver/server cursor behavior may buffer; verify implementation.
8. Updates/deletes to old IDs.
9. Resolve timestamp ties deterministically.
10. Avoid progress recorded before data is durable.
11. Replica has not applied all primary changes yet.
12. Evidence output matches defined source scope and rules.

---

# 15. Day 33 self-check

- [ ] Connect and verify correct database/user.
- [ ] Execute explicit SQL and parameterized filter.
- [ ] Explain fetchone/fetchmany/fetchall.
- [ ] Process output in batches.
- [ ] Write CSV without collecting all rows.
- [ ] Reconcile source/output count and keys.
- [ ] Explain ID/timestamp watermark limitations.
- [ ] Describe source safety and replica lag.
- [ ] Demonstrate a crash/replay plan.
- [ ] Defend the database extractor project.

**Remember:** Database ingestion is not just SELECT. It includes source protection, bounded reading, consistency, durable progress, replay and evidence.


# 16. More guided execution cells (do not skip)

## Cell A — Compare SQL count with Python count

Run in SQL client:
```sql
SELECT COUNT(*) FROM orders;
```
Then run in Python:
```python
cursor.execute("SELECT COUNT(*) FROM orders")
source_count = cursor.fetchone()[0]
print("Source count:", source_count)
```
The two values must match when they use the same database/table and no filter. If they differ, verify database connection, permissions, transaction snapshot and query text.

## Cell B — Explain a batch boundary

For a table with 23 rows and batch_size=10, the loop returns 10,10,3,then 0. The empty batch is the stop signal; the final 3 rows are valid and must be written.

| Fetch call | Returned rows | Running total |
|---|---:|---:|
| 1 | 10 | 10 |
| 2 | 10 | 20 |
| 3 | 3 | 23 |
| 4 | 0 | 23 (stop) |

## Cell C — Write batch progress safely

For each batch, write rows to a temporary output and track batch number and row count. A checkpoint must represent only data that has been durably committed. If the file/database write is not durable, do not move the watermark.

## Cell D — Demonstrate a duplicate replay

Take a 3-row batch and write it twice to a plain append CSV. The file now has 6 rows: this demonstrates why append is not idempotent. For a daily full extract, write to a temporary file and replace only after validation. For database targets, use a stable key and a controlled upsert/merge.

## Cell E — Explain a join multiplication risk

Suppose one order has two order_items and three payment_attempts. Joining both child tables by order_id can create 2 × 3 = 6 rows for that order. Summing order amount after this join can multiply the amount. Inspect table grain, aggregate children first or query at the correct grain, and reconcile.

## Cell F — Verify first/last key

For a deterministic append-only extract, inspect MIN(order_id), MAX(order_id), count and distinct count. A range alone does not prove no gaps; compare expected key set or source/target reconciliation when required.

## Cell G — Use query plan safely

Ask the DBA or use EXPLAIN to understand scans/filters. EXPLAIN ANALYZE executes the query; do not casually run it on expensive production queries. Check index use and row estimates against actual workload.

## Cell H — Empty table

A source count of zero should produce a header-only output or an explicitly defined empty result. The pipeline must still record run date, source, zero rows and status. Do not treat empty fetch as a crash.

# 17. Practical decision table

| Situation | Junior engineer action | Evidence |
|---|---|---|
| Small source | fetchall may be acceptable | measured size/count |
| Large source | bounded batches/server cursor | memory and batch metrics |
| Append-only table | ID boundary may work | source guarantees append-only |
| Mutable table | updated_at/CDC/change feed | update/delete test |
| Tied timestamps | compound cursor | same-time test rows |
| Replica source | monitor lag | applied position/lag metric |
| Partial output | temp + validation | output count/hash |
| Rerun | replay-safe write | duplicate test |
| Query slow | plan/filters/index review | explain and duration |

# 18. Additional student exercise

Design extraction for 4 million orders updated throughout the day. Write a short plan that answers:
- Which source change mechanism would you request and why?
- How do you bound the initial load?
- How do you avoid missing changes during initial snapshot handoff?
- How do you fetch/write batches?
- How do you capture deletes?
- What is checkpoint order?
- What reconciliation evidence is feasible?
- How do you protect the source database?

**Suggested reasoning:** Do not assume ID alone captures updates. Ask for snapshot/CDC handoff or documented updated timestamp/change feed. Use bounded reads, durable staging, idempotent merge, source-load controls, delete/tombstone policy and key/count/aggregate reconciliation. Exact architecture depends on DB and provider capabilities.
