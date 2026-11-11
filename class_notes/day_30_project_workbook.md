# Day 30 Project Workbook — Build, Run, Test and Recover

This workbook is designed so a student can complete the project without guessing missing steps. Follow the milestones in order. Use the actual TechStore database and adapt column names only after inspecting its schema.

## Project outcome

A repeatable daily CSV extraction, validated before publication, with a run summary and safe rerun behavior.

## Step 0 — Confirm prerequisites

- PostgreSQL service is running.
- TechStore database and orders table exist (Day 19/20).
- Python and Jupyter/VS Code are available.
- Psycopg 3 is installed in the notebook's Python environment.
- You know the source table's actual column names and timestamp timezone.
- You can connect using a read-only source user (recommended).

**Do not proceed by inventing a database schema.** If your Day 19/20 schema differs, map the real columns in the query.

## Step 1 — Make project folders

In Windows CMD, navigate to your course practice location and run:

```bat
mkdir day30_etl_project
cd day30_etl_project
mkdir output
mkdir notebook
```

Create a Jupyter notebook in notebook folder named `day30_daily_extract.ipynb`.

Expected structure:
```text
day30_etl_project
|-- output
|-- notebook
    |-- day30_daily_extract.ipynb
```

## Step 2 — Set connection environment variables

In the CMD window that will launch Jupyter:

```bat
set PGHOST=localhost
set PGPORT=5432
set PGDATABASE=techstore
set PGUSER=postgres
set PGPASSWORD=YOUR_LOCAL_PASSWORD
```

Replace local values. Never commit this password or share a notebook containing it.

Start Jupyter from this same terminal. If Jupyter was already running, stop it and launch it again after setting variables.

## Step 3 — Notebook cell: imports and configuration

```python
import os
import csv
import psycopg
from datetime import datetime, date, time, timedelta

business_date = date(2026, 11, 10)
start_time = datetime.combine(business_date, time.min)
end_time = datetime.combine(business_date + timedelta(days=1), time.min)

output_path = f"../output/orders_{business_date.isoformat()}.csv"

print("Business date:", business_date)
print("Start:", start_time)
print("End (exclusive):", end_time)
print("Output:", output_path)
```

**Expected output**
```text
Business date: 2026-11-10
Start: 2026-11-10 00:00:00
End (exclusive): 2026-11-11 00:00:00
Output: ../output/orders_2026-11-10.csv
```

Explanation:
- date stores the business date, not a time.
- timedelta(days=1) calculates the next day safely, including month/year boundaries.
- isoformat produces a stable YYYY-MM-DD filename component.
- The output path is relative to the notebook working directory; verify it in your environment.

## Step 4 — Open connection and inspect schema

```python
connection = psycopg.connect(
    host=os.environ["PGHOST"],
    port=os.environ["PGPORT"],
    dbname=os.environ["PGDATABASE"],
    user=os.environ["PGUSER"],
    password=os.environ["PGPASSWORD"]
)

cursor = connection.cursor()
cursor.execute("""
SELECT column_name
FROM information_schema.columns
WHERE table_name = 'orders'
ORDER BY ordinal_position
""")

print([row[0] for row in cursor.fetchall()])
```

Expected: a list of the actual orders table columns. Confirm the five extraction columns are present. If not, stop and adapt the query based on your real schema.

## Step 5 — Execute the bounded extraction

```python
query = """
SELECT order_id, customer_id, created_at, status, total_amount
FROM orders
WHERE created_at >= %s
  AND created_at < %s
ORDER BY order_id
"""

cursor.execute(query, (start_time, end_time))
column_names = [item.name for item in cursor.description]
rows = cursor.fetchall()

print("Columns:", column_names)
print("Source rows fetched:", len(rows))
```

**Expected output shape**
```text
Columns: ['order_id', 'customer_id', 'created_at', 'status', 'total_amount']
Source rows fetched: N
```

N is the count for the actual selected date. If the source uses timestamptz, use timezone-aware boundaries and document the business timezone.

## Step 6 — Convert result rows to dictionaries

```python
records = []

for row in rows:
    records.append(dict(zip(column_names, row)))

print("Records prepared:", len(records))
print("First record:", records[0] if records else None)
```

Expected: Records prepared equals Source rows fetched. First record is a dictionary with the selected column names and values, or None for an empty day.

## Step 7 — Write a temporary file first

```python
temporary_path = output_path + ".tmp"

with open(temporary_path, "w", newline="", encoding="utf-8") as file:
    writer = csv.DictWriter(file, fieldnames=column_names)
    writer.writeheader()
    writer.writerows(records)

print("Temporary file written:", temporary_path)
```

Why temporary? A crash during write should not leave a half-written file under the final published name. For production, ensure temp and final are on the same filesystem and use the platform's atomic rename/replace semantics.

## Step 8 — Reopen the CSV and count rows

```python
with open(temporary_path, "r", newline="", encoding="utf-8") as file:
    reader = csv.DictReader(file)
    csv_rows = list(reader)

print("CSV data rows:", len(csv_rows))
print("CSV header:", reader.fieldnames)
```

Expected:
- CSV data rows equals len(records).
- Header equals column_names.
- Empty source produces zero data rows but still a header.

## Step 9 — Validate keys and boundaries

```python
order_ids = [record["order_id"] for record in records]
duplicate_count = len(order_ids) - len(set(order_ids))

outside_window_count = 0
for record in records:
    if not (start_time <= record["created_at"] < end_time):
        outside_window_count += 1

print("Duplicate order IDs:", duplicate_count)
print("Rows outside window:", outside_window_count)
print("Count match:", len(records) == len(csv_rows))
```

Expected:
```text
Duplicate order IDs: 0
Rows outside window: 0
Count match: True
```

If any expected check fails, do not publish the file. Investigate source key, query, timestamp semantics or write process.

## Step 10 — Publish only after validation

```python
import os

checks_passed = (
    len(records) == len(csv_rows)
    and duplicate_count == 0
    and outside_window_count == 0
)

if checks_passed:
    os.replace(temporary_path, output_path)
    run_status = "SUCCESS"
else:
    run_status = "FAILED_VALIDATION"

print("Run status:", run_status)
```

`os.replace` replaces an existing destination. In a real production workflow, output versioning, permissions and atomicity must be checked for the chosen filesystem/object store. Never overwrite a valuable deliverable without a defined policy.

## Step 11 — Build the run summary

```python
run_summary = {
    "run_date": business_date.isoformat(),
    "source_rows": len(records),
    "csv_rows": len(csv_rows),
    "duplicate_order_ids": duplicate_count,
    "outside_window_rows": outside_window_count,
    "output_path": output_path,
    "status": run_status
}

print(run_summary)
```

Expected shape:
```text
{'run_date': '2026-11-10',
 'source_rows': N,
 'csv_rows': N,
 'duplicate_order_ids': 0,
 'outside_window_rows': 0,
 'output_path': '../output/orders_2026-11-10.csv',
 'status': 'SUCCESS'}
```

N must be the same observed count. Do not write a fake value.

## Step 12 — Close database resources

```python
cursor.close()
connection.close()
print("Connection and cursor closed")
```

If an earlier cell fails, run cleanup before retrying. In production use try/finally or context managers to guarantee cleanup.

## Step 13 — Failure drills

### Drill A: Wrong password
Change password temporarily to an invalid value. Observe authentication error. Restore correct value; rerun connection cell. Never print the password.

### Drill B: Wrong date
Choose a date with no orders. Confirm output has header and zero rows. Decide whether zero is acceptable according to the business rule.

### Drill C: Duplicate key simulation
Create a duplicate order ID in a local test fixture or deliberately duplicate a record in a Python list. Verify duplicate_count becomes nonzero and publication is blocked. Do not alter production source data for this drill.

### Drill D: Interrupt write
Stop execution while writing a large test file. Confirm final published file is not replaced by a partial temp file. Remove abandoned temp file after inspection.

### Drill E: Rerun same business date
Run again. Confirm the final file is deterministic and contains no appended duplicate rows. Compare row count and key set.

## Project acceptance criteria

- [ ] Correct business-date range and timezone documented.
- [ ] Explicit source columns used.
- [ ] Connection credentials outside source/notebook.
- [ ] SQL parameters passed safely.
- [ ] Source count, fetched count and CSV count reconciled.
- [ ] Unique key and date-boundary checks pass.
- [ ] Temp file validated before publication.
- [ ] Empty-day behavior documented.
- [ ] Rerun does not duplicate output.
- [ ] Resources closed and run summary recorded.
- [ ] Student can explain every project cell.

**Project complete means all acceptance criteria pass—not merely that a CSV appeared.**
