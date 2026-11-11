# Day 33 Project Workbook — TechStore Database Extractor

## Project goal
Read orders from PostgreSQL in bounded batches, write a CSV extract and prove correctness without loading the entire source into one Python list.

## Step 0 — Prerequisites
- TechStore database/table exists.
- Python/Psycopg 3 installed.
- Environment variables set (PGHOST, PGPORT, PGDATABASE, PGUSER, PGPASSWORD).
- Prefer a read-only database account.
- Inspect actual column names before running.

## Step 1 — Folder
CMD:
```bat
mkdir day33_database_ingestion
cd day33_database_ingestion
mkdir output notebook
```
Create notebook `notebook/day33_database_extractor.ipynb`.

## Step 2 — Connect and verify source
```python
import os, csv, psycopg
from pathlib import Path

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
Confirm the expected database and account. Do not proceed if connected to the wrong source.

## Step 3 — Record source count and upper boundary
```python
cursor.execute("SELECT COUNT(*), MAX(order_id) FROM orders")
source_count, upper_id = cursor.fetchone()
print("Source count:", source_count)
print("Upper ID:", upper_id)
```
These values are actual source values. For an empty table, MAX is None.

## Step 4 — Extract bounded query
```python
query = """
SELECT order_id, customer_id, created_at, status, total_amount
FROM orders
WHERE order_id <= %s
ORDER BY order_id
"""
cursor.execute(query, (upper_id,))
column_names = [column.name for column in cursor.description]
```
If upper_id is None (empty table), skip the extraction and create a header-only output. Never pass None as if it were a valid upper boundary.

## Step 5 — Batch write
```python
temporary_path = Path("output/orders_extract.csv.tmp")
final_path = Path("output/orders_extract.csv")
temporary_path.parent.mkdir(parents=True, exist_ok=True)

total_written = 0
batch_count = 0

with open(temporary_path, "w", newline="", encoding="utf-8") as file:
    writer = csv.writer(file)
    writer.writerow(column_names)

    while True:
        batch = cursor.fetchmany(500)
        if not batch:
            break
        writer.writerows(batch)
        total_written += len(batch)
        batch_count += 1

print("Batches:", batch_count)
print("Rows written:", total_written)
```
For N rows, batch count is approximately ceil(N/500), except N=0 yields zero batches. The final batch may be smaller.

## Step 6 — Verify output
```python
with open(temporary_path, "r", newline="", encoding="utf-8") as file:
    output_rows = list(csv.DictReader(file))

output_count = len(output_rows)
print("Source count:", source_count)
print("Output count:", output_count)
print("Count match:", source_count == output_count)
```
Expected Count match True when query scope matches count query and source consistency is appropriate. Concurrent updates/transaction snapshots can affect comparison; document the consistency method.

## Step 7 — Check keys
For a small practice table:
```python
output_ids = [int(row["order_id"]) for row in output_rows]
print("Unique IDs:", len(set(output_ids)))
print("Output IDs:", len(output_ids))
```
For large data, don't load all IDs into memory just to test uniqueness; use source/target constraints, SQL aggregation or scalable validation.

## Step 8 — Publish
Only after validation:
```python
import os
os.replace(temporary_path, final_path)
print("Published:", final_path)
```
Replace semantics depend on filesystem. A production pipeline needs an explicit overwrite/versioning policy.

## Step 9 — Run summary
```python
run_summary = {
    "source_count": source_count,
    "rows_written": total_written,
    "batch_count": batch_count,
    "upper_id": upper_id,
    "count_match": source_count == output_count,
    "status": "SUCCESS" if source_count == output_count else "FAILED"
}
print(run_summary)
```
Do not set success if other validations failed.

## Step 10 — Failure drills
- Query timeout: stop and inspect source/query plan; don't infinite retry.
- Connection loss: close/reconnect and replay from safe point.
- Interrupted file: verify temp file; do not publish partial output.
- Same run twice: confirm no append duplication.
- Empty source: header-only file and zero count.
- Old ID update: explain why ID watermark misses it.
- Timestamp tie: use compound cursor.
- Replica lag: verify source applied position/lag.

## Acceptance criteria
- [ ] Correct DB/user verified.
- [ ] Explicit columns and parameter placeholders used.
- [ ] Read-only source access.
- [ ] Batches processed and counted.
- [ ] Output reopened and count checked.
- [ ] Empty source handled.
- [ ] Temp output validated before publication.
- [ ] Watermark limitations explained.
- [ ] Rerun/failure behavior tested.
- [ ] Student can defend source consistency and checkpoint ordering.
