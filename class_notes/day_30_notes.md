# Day 30: ETL and Data Extraction — Beginner Notebook Class

**Phase:** 4 — Data Processing and Pipelines  
**Time:** 2–3 hours  
**Project:** TechStore Daily Orders Extractor  
**Prerequisites:** Day 19–29 database/SQL concepts; basic Python variables, loops and files  
**Environment:** PostgreSQL + Python + Jupyter Notebook or VS Code  
**Student promise:** Follow each cell in order. Do not jump to the final project code.

---

# 0. What you will build today

At the end, you will build a small pipeline that reads order records from a source database, selects only the required rows, saves an extract, and checks whether the extract is correct.

The flow is:

`Source PostgreSQL table -> Extract SQL -> Python receives rows -> CSV file -> Validation -> Run summary`

Imagine TechStore sells products online. The operations database stores every order. The analytics team needs yesterday's orders for a daily sales report. Copying rows manually is slow and error-prone. A pipeline repeats the same controlled steps.

## 0.1 Outcomes

By the end, you can:
- explain ETL and extraction in plain language;
- distinguish source, extraction, transformation and destination;
- run a bounded SQL extraction;
- connect Python to PostgreSQL safely;
- fetch a small result and inspect its Python types;
- write an extract file;
- validate row counts and keys;
- explain full extraction versus incremental extraction;
- identify common failures and recover safely.

## 0.2 Vocabulary before code

| Word | Beginner meaning | TechStore example |
|---|---|---|
| Source | Where data originally lives | PostgreSQL orders table |
| Extract | Copy a selected set of records out | Orders created on 11 Nov |
| Transform | Change/clean/derive values under rules | Standardize status |
| Load | Write data into another destination | Insert into analytics table |
| Pipeline | Repeatable connected steps | Extract -> validate -> publish |
| Batch | A manageable group of records | 500 orders at a time |
| Watermark | A saved boundary for progress | Last safely processed order ID |
| Reconciliation | Compare evidence to prove correctness | Source count equals file count |

**Important:** ETL is Extract, Transform, Load. ELT is Extract, Load, Transform. They differ in where transformation happens. We will not pretend every pipeline must use the same architecture.

---

# 1. Understand the business request

A manager says:

> “Every morning, give me yesterday's orders in a CSV. Include order ID, customer ID, order date, status and amount. Do not include today's orders. Tell me how many rows were exported.”

Before coding, translate this into a contract.

| Requirement | Technical interpretation |
|---|---|
| Yesterday's orders | Filter by a defined date/time boundary |
| Include five fields | Explicit SELECT columns |
| Do not include today's orders | End boundary is exclusive |
| CSV output | A named file with header |
| Count exported | Record count before/after writing |
| Prove correctness | Compare source query count and output count |

### Why use an exclusive end boundary?

Suppose yesterday is 2026-11-10. The safe time range is:

`created_at >= 2026-11-10 00:00:00`  
`created_at < 2026-11-11 00:00:00`

This includes every time during the day and excludes exactly midnight of the next day. Avoid writing `created_at <= 23:59:59`; timestamps can include fractions such as 23:59:59.500.

**Timezone decision:** This lesson uses the database's documented business timezone. In a real system, agree whether reporting day means UTC or a business timezone (for example Asia/Kolkata), then convert boundaries consistently.

---

# 2. Inspect the source before extracting

Do not start by writing a pipeline. First understand the source table.

## 2.1 Open PostgreSQL

1. Open pgAdmin or your SQL client.
2. Connect to the TechStore database created in Day 19/20.
3. Open Query Tool.
4. Execute each SQL cell separately.
5. Compare your output with the expected output shown below.

If you do not have the TechStore database, complete the Day 19/20 setup first. Do not invent a source table.

## 2.2 Inspect table columns

```sql
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'orders'
ORDER BY ordinal_position;
```

**What this does:** `information_schema.columns` is PostgreSQL metadata describing columns. The WHERE clause selects metadata for the orders table. ORDER BY makes the display predictable.

Your output should list the actual columns from your Day 19/20 schema. If names differ, use your real schema and update the later query—never silently assume a column exists.

## 2.3 Inspect a few rows

```sql
SELECT *
FROM orders
ORDER BY order_id
LIMIT 5;
```

- `SELECT * ` means every column. It is useful for exploration, not usually ideal for a stable pipeline.
- `ORDER BY order_id` gives deterministic order if order_id is unique.
- `LIMIT 5` keeps the inspection small.

**Checkpoint:** Write down the order ID, customer ID, timestamp, status and amount column names. The following extraction uses those names.

---

# 3. Your first extraction query (SQL only)

## 3.1 Select explicit columns

```sql
SELECT order_id, customer_id, created_at, status, total_amount
FROM orders
ORDER BY order_id;
```

This is an extraction query. It does not update or delete the source.

### Read the query aloud

- SELECT: “I want these columns.”
- FROM: “Read them from this table.”
- ORDER BY: “Return rows in a predictable order.”

### Why not SELECT * in production?

A source table may gain a new column. A pipeline using `*` can unexpectedly receive extra fields, increase data volume, or break downstream assumptions. Explicit columns make the contract visible.

## 3.2 Filter one business day

Replace the date literals with the date you are practicing.

```sql
SELECT order_id, customer_id, created_at, status, total_amount
FROM orders
WHERE created_at >= TIMESTAMP '2026-11-10 00:00:00'
  AND created_at <  TIMESTAMP '2026-11-11 00:00:00'
ORDER BY order_id;
```

**Expected result:** Only rows whose created_at is inside that interval. The exact rows depend on your TechStore seed data. Do not copy an expected count from a different dataset.

### Check the boundary yourself

```sql
SELECT
  COUNT(*) AS extracted_rows,
  MIN(created_at) AS first_order_time,
  MAX(created_at) AS last_order_time
FROM orders
WHERE created_at >= TIMESTAMP '2026-11-10 00:00:00'
  AND created_at <  TIMESTAMP '2026-11-11 00:00:00';
```

Interpret:
- COUNT tells how many records match.
- MIN is the earliest timestamp in the result.
- MAX is the latest timestamp in the result.
- If no rows match, COUNT is 0 and MIN/MAX are NULL. That is a valid empty result—not automatically a failure.

---

# 4. Python environment and database driver

Python needs a driver to speak PostgreSQL's protocol. We use Psycopg 3.

## 4.1 Install

In the terminal (not inside a SQL query window):

```bash
python -m pip install "psycopg[binary]"
```

If using Jupyter and its kernel points to another Python environment, install into that kernel's environment. After installation, restart the notebook kernel if the import still fails.

## 4.2 Verify installation

Notebook cell:

```python
import psycopg

print("Psycopg version:", psycopg.__version__)
```

Expected output shape:

```text
Psycopg version: 3.x.x
```

The exact patch version depends on the installed package. This is intentionally not a fixed output.

**If ModuleNotFoundError:** The package was installed in a different Python environment. Check which Python runs the notebook, install into that environment, restart and retry.

---

# 5. Connect Python to PostgreSQL

## 5.1 Connection settings

Use environment variables for credentials. Do not place real passwords inside a notebook or commit them to Git.

For a local practice database, set these in your terminal before launching Jupyter/VS Code:

Windows CMD:
```bat
set PGHOST=localhost
set PGPORT=5432
set PGDATABASE=techstore
set PGUSER=postgres
set PGPASSWORD=YOUR_LOCAL_PASSWORD
```

Replace the database/user/password with your own local values. These variables apply to that CMD window and processes launched from it.

## 5.2 First connection cell

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

print("Database connection opened")
```

### Explain every piece

- `import os`: provides access to environment variables.
- `import psycopg`: imports the PostgreSQL driver.
- `os.environ["PGHOST"]`: reads the variable; if absent, Python raises KeyError instead of silently connecting to an unknown host.
- `psycopg.connect(...)`: opens a connection using the settings.
- `connection`: stores the connection object for later cells.

Expected:
```text
Database connection opened
```

This message proves the connection call returned successfully. It does **not** prove that the correct table/data exists.

### Connection troubleshooting

| Error | Likely cause | What to check |
|---|---|---|
| KeyError PGHOST | Environment variable absent | Set variables in parent terminal; restart notebook |
| connection refused | PostgreSQL not running/wrong port | Start service; verify port |
| password authentication failed | Wrong user/password | Check local credentials; don't print password |
| database does not exist | PGDATABASE incorrect | Use actual created database name |
| timeout | Host/network issue | Check host, firewall, network and server |

---

# 6. Execute SQL from Python

## 6.1 Create a cursor

```python
cursor = connection.cursor()
print("Cursor created")
```

A cursor is the object used to execute SQL and read query results through this connection.

## 6.2 Execute a tiny query

```python
cursor.execute("SELECT 1 AS health_check")
row = cursor.fetchone()
print(row)
```

Expected:
```text
(1,)
```

Why a tuple? A database row is returned as a sequence of column values. Even one column is represented as a one-item tuple.

`fetchone()` returns one row. It does not return the entire result set.

## 6.3 Execute the extraction query

```python
sql = """
SELECT order_id, customer_id, created_at, status, total_amount
FROM orders
WHERE created_at >= TIMESTAMP '2026-11-10 00:00:00'
  AND created_at <  TIMESTAMP '2026-11-11 00:00:00'
ORDER BY order_id
"""

cursor.execute(sql)
first_row = cursor.fetchone()
print(first_row)
```

The first row depends on the actual seed data. If no order falls on that date, `first_row` is `None`.

**Checkpoint:** Run the same SQL in pgAdmin. The first row should match Python's first row. If it doesn't, compare database name, timezone, query text and ordering.

---

# 7. Fetch all rows (small practice data only)

```python
cursor.execute(sql)
rows = cursor.fetchall()

print("Rows fetched:", len(rows))
print("First row:", rows[0] if rows else "No matching rows")
```

- `fetchall()` reads all remaining result rows into Python memory.
- `len(rows)` counts the returned records.
- The conditional expression avoids indexing an empty list.

**Memory warning:** `fetchall()` is acceptable for a small classroom table. A production table with millions of rows can exhaust memory. Day 33 and Day 36 teach bounded batches.

## 7.1 Inspect data types

```python
if rows:
    print("Order ID type:", type(rows[0][0]))
    print("Timestamp type:", type(rows[0][2]))
    print("Amount type:", type(rows[0][4]))
```

Typical mapping:
- PostgreSQL integer -> Python int
- timestamp -> Python datetime
- numeric/decimal -> Python Decimal
- SQL NULL -> Python None

Do not assume amount is float. Decimal is valuable for exact financial calculations.

---

# 8. Make the extraction readable

Tuple positions are easy to confuse. Ask the cursor for column names.

```python
column_names = [column.name for column in cursor.description]
print(column_names)
```

Expected:
```text
['order_id', 'customer_id', 'created_at', 'status', 'total_amount']
```

This list comes from the SELECT column order. If you change SELECT, this list changes too.

Create dictionaries for a small result:

```python
records = []

for row in rows:
    record = dict(zip(column_names, row))
    records.append(record)

print(records[0] if records else "No records")
```

Step-by-step:
1. `for row in rows` visits each tuple.
2. `zip(column_names, row)` pairs each name with its value.
3. `dict(...)` creates a dictionary for that row.
4. `append` adds the dictionary to the list.

This is a learning step. For large data, do not build a giant list; stream/batch to output.

---

# 9. Write a CSV extract

## 9.1 Why CSV?

CSV is a plain text table format. It is easy to inspect and exchange, but it does not preserve all database types automatically. A timestamp or Decimal becomes text representation when written.

## 9.2 Write the file

```python
import csv

output_path = "orders_2026-11-10.csv"

with open(output_path, "w", newline="", encoding="utf-8") as file:
    writer = csv.DictWriter(file, fieldnames=column_names)
    writer.writeheader()
    writer.writerows(records)

print("CSV created:", output_path)
print("Rows written:", len(records))
```

Explain:
- `import csv`: standard Python CSV module.
- `open(..., "w")`: creates/overwrites the output file. Do not use this mode on a valuable file without knowing it overwrites.
- `newline=""`: recommended for CSV to avoid extra blank lines on some systems.
- `encoding="utf-8"`: supports broad text characters.
- `DictWriter`: maps dictionary keys to CSV columns.
- `writeheader()`: writes the column names as first row.
- `writerows(records)`: writes each dictionary.
- `with`: closes the file even if an error occurs.

Expected:
```text
CSV created: orders_2026-11-10.csv
Rows written: <number matching the SQL filter>
```

The count is dataset-dependent. Replace the placeholder with the number you actually observed.

## 9.3 Inspect the CSV

Open the file in VS Code or Excel. Verify:
- first row is the header;
- each data row has five fields;
- first/last order IDs match the SQL output;
- no order from the next day appears;
- file row count (excluding header) equals `len(records)`.

---

# 10. Validate before publishing

A file existing does not mean it is correct.

## 10.1 Count invariant

`source query count = Python fetched count = CSV data row count`

Run SQL COUNT again. Compare it with Python and file.

## 10.2 Check key uniqueness

```python
order_ids = [record["order_id"] for record in records]

print("Unique order IDs:", len(set(order_ids)))
print("Total order IDs:", len(order_ids))
print("Duplicate IDs:", len(order_ids) - len(set(order_ids)))
```

Expected for a unique order table:
```text
Unique order IDs: <same count>
Total order IDs: <same count>
Duplicate IDs: 0
```

If duplicates appear, stop publication and investigate the source query/key. Do not silently drop duplicates unless the business rule says how.

## 10.3 Date boundary check

```python
from datetime import datetime

start = datetime(2026, 11, 10, 0, 0, 0)
end = datetime(2026, 11, 11, 0, 0, 0)

outside_window = [
    record for record in records
    if not (start <= record["created_at"] < end)
]

print("Rows outside requested window:", len(outside_window))
```

Expected:
```text
Rows outside requested window: 0
```

This assumes database timestamps and Python boundaries use compatible timezone semantics. If the source column is timezone-aware, use timezone-aware boundaries and explicitly agree on timezone.

---

# 11. Full extraction vs incremental extraction

## Full extraction
Read every record each run. Simple but increasingly expensive.

## Incremental extraction
Read only new/changed records since a safe boundary. Efficient but requires reliable change tracking.

| Strategy | Example | Main risk |
|---|---|---|
| Full load | SELECT all orders | Source load and repeated volume |
| ID watermark | order_id > last_id | Misses updates/deletes to old IDs |
| Updated timestamp | updated_at > last_time | Timestamp ties/late changes |
| Compound cursor | updated_at + unique ID | More careful boundary handling |
| CDC | database change log | Operational complexity/permissions |

**Mentor point:** An ID watermark is not automatically a correct incremental strategy for mutable records. An old order can be updated without receiving a new ID.

---

# 12. Beginner debugging clinic

## Problem A: No rows returned
1. Run COUNT query in SQL client.
2. Check date values and timezone.
3. Confirm source database/table.
4. Treat an empty day as valid if business allows it.
5. Do not change filters merely to force rows.

## Problem B: CSV has extra blank rows
Use `newline=""` when opening the file. Inspect the file in a text editor.

## Problem C: Amount looks different in Excel
CSV stores text representations; Excel may format/round display. Compare raw text and database Decimal before concluding data changed.

## Problem D: File overwritten
The "w" mode overwrites. Use a controlled output directory and run ID/date naming; protect important outputs.

## Problem E: Python row count differs from SQL
Check that both execute the same query, database, timezone and boundary; ensure you didn't fetch only one row; inspect whether a cursor was already partially consumed.

---

# 13. Mini-project: TechStore Daily Orders Extractor

## Requirement
Produce a daily CSV extract and a run summary. This is a guided project, not a copy-only exercise.

### Project folders

Create:

```text
day30_etl_project/
├── output/
│   └── orders_YYYY-MM-DD.csv
└── notebook/
    └── day30_daily_extract.ipynb
```

### Milestone 1 — Define the run
- Choose business date.
- Write start/end boundaries.
- Identify required source columns.
- Decide output filename.

### Milestone 2 — Extract
- Connect with environment variables.
- Execute explicit-column query.
- Fetch rows.
- Record source count.

### Milestone 3 — Publish file
- Write header and records.
- Verify file exists and can be reopened.
- Count CSV data rows.

### Milestone 4 — Validate
- Source count equals fetched count.
- Fetched count equals CSV count.
- Unique order IDs equal row count.
- Every timestamp is in the requested interval.
- No secrets appear in notebook or logs.

### Milestone 5 — Run summary
Create a summary dictionary:

```python
run_summary = {
    "business_date": "2026-11-10",
    "source_rows": len(records),
    "file_rows": len(records),
    "duplicate_order_ids": len(order_ids) - len(set(order_ids)),
    "outside_window_rows": len(outside_window),
    "output_file": output_path,
    "status": "SUCCESS"
}

print(run_summary)
```

Expected shape (counts depend on seed data):

```text
{'business_date': '2026-11-10',
 'source_rows': <N>,
 'file_rows': <N>,
 'duplicate_order_ids': 0,
 'outside_window_rows': 0,
 'output_file': 'orders_2026-11-10.csv',
 'status': 'SUCCESS'}
```

**Do not set SUCCESS if a validation failed.** In a real pipeline, status should be calculated from validation results rather than typed manually.

### Milestone 6 — Clean up resources

```python
cursor.close()
connection.close()

print("Database resources closed")
```

Closing releases resources. In production, use a context manager or try/finally so cleanup also happens after exceptions.

---

# 14. Practice exercises with answer guide

1. Change the business date to another day. Which two boundary values must change?
2. Why is end boundary exclusive?
3. What does fetchone return when no row exists?
4. Why is fetchall unsafe for a very large table?
5. What does cursor.description contain?
6. Why does CSV not preserve database types?
7. What is one reason a unique order ID check can fail?
8. What could an ID watermark miss?
9. Why should credentials not be in source code?
10. What evidence is needed before saying the extract succeeded?

## Answers
1. Start at that date's midnight; end at next date's midnight.
2. It includes all fractional timestamps on the day and excludes the next day's midnight.
3. None.
4. It loads all remaining rows into memory.
5. Metadata for selected columns, including their names.
6. CSV is text; type interpretation occurs when another application reads it.
7. Duplicate source rows, join multiplication or incorrect key assumption.
8. Updates/deletes to older IDs.
9. Secrets can leak through Git, sharing and logs; use environment/secret storage.
10. Matching counts, unique keys, boundary check, readable file and a run summary.

---

# 15. Interview preparation — concepts you must explain

- Explain ETL vs ELT with an example.
- Explain extraction and source-system safety.
- Explain full load vs incremental load.
- Explain watermark and its limitations.
- Explain why explicit columns and deterministic ordering matter.
- Explain why fetchall is acceptable for small data but risky at scale.
- Explain source count, output count and reconciliation.
- Explain how you would recover if the process crashed after writing the CSV.
- Explain why successful connection is not proof of successful pipeline.
- Explain what makes an extraction pipeline production-ready.

**Interview response structure:** business requirement -> extraction design -> correctness checks -> failure/recovery -> operational safety. Avoid giving only a definition.

---

# 16. Day 30 final self-check

You are ready to move on only if you can do all of these without copying a completed solution blindly:

- [ ] Explain ETL and extraction using TechStore.
- [ ] Inspect the actual source schema.
- [ ] Run a date-bounded SQL query.
- [ ] Connect Python to PostgreSQL using environment variables.
- [ ] Explain connection, cursor, execute and fetch.
- [ ] Produce a CSV with a header.
- [ ] Verify counts, keys and time boundaries.
- [ ] Explain full versus incremental and watermark risk.
- [ ] Diagnose at least two deliberate failures.
- [ ] Defend your project design in an interview.

**Remember:** The pipeline is not complete when the file is created. It is complete when the data is correct, verified, safely published and recoverable.
