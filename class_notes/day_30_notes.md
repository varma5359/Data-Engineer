# Day 30: ETL and Data Extraction Patterns

**Phase:** 4 — Data Processing and Pipelines  
**Syllabus topic:** ETL and Data Extraction Patterns  
**Mini-project:** Incremental Extraction Pipeline  
**Learning time:** 2–3 hours  
**Level:** Complete beginner to junior Data Engineer  
**Tools:** Python, SQL, PostgreSQL (examples use PostgreSQL syntax where noted)

---

# 1. What are we learning today?

In Phase 3, you learned how to design databases, write SQL queries, use transactions and connect Python to PostgreSQL.

Now we move from querying data to building a pipeline that regularly collects data from a source system.

A company does not want an employee to manually copy yesterday's orders into a reporting database every morning. It wants a repeatable process that can collect new or changed records, validate them and send them to the next system.

That process begins with extraction.

## 1.1 Learning outcomes

By the end of this lesson, you should be able to:

1. Explain ETL and the role of extraction.
2. Identify source systems and extraction requirements.
3. Distinguish full extraction from incremental extraction.
4. Explain snapshot, timestamp and key-based extraction.
5. Explain what a watermark is.
6. Build a simple watermark-based extraction query.
7. Understand inclusive and exclusive time boundaries.
8. Handle records that arrive late.
9. Explain why source ordering and stable keys matter.
10. Understand extraction checkpoints.
11. Separate extraction from transformation and loading.
12. Build a small incremental extraction pipeline.
13. Verify that the pipeline does not skip records.
14. Explain common extraction failures in an interview.

## 1.2 Today's pipeline

We will use an online retail company's order system.

```text
Retail order database
        |
        | Extract new/changed orders
        v
Python extraction process
        |
        | Validate extracted rows
        v
Staging / target database
        |
        | Reconcile counts and totals
        v
Checkpoint updated
```

Today's focus is the first major responsibility: **extracting the correct source records reliably**.

---

# 2. First understand the business problem

Imagine an online store receives 20,000 orders every day.

Its operational database stores all orders:

| order_id | customer | amount | status | updated_at |
|---:|---|---:|---|---|
| 1001 | Asha | 1500.00 | PAID | 2026-11-01 09:10 |
| 1002 | Ravi | 2800.00 | SHIPPED | 2026-11-01 10:15 |
| 1003 | Meena | 900.00 | PENDING | 2026-11-02 08:30 |
| 1004 | Kiran | 4500.00 | PAID | 2026-11-03 11:45 |

The analytics team needs order data in a reporting database.

A beginner might run this query every day:

```sql
SELECT *
FROM orders;
```

This returns all rows every time.

If the table contains 10 million records, repeatedly extracting all 10 million rows may waste:
- Network bandwidth.
- Database CPU and I/O.
- Python memory.
- Processing time.
- Cloud storage and compute cost.

A Data Engineer asks a better question:

**Can I extract only the records that are new or have changed since the last successful run?**

That is the reason we study extraction patterns.

---

# 3. What is ETL?

ETL stands for:

- **E — Extract:** collect data from a source.
- **T — Transform:** clean, standardize, validate or calculate values.
- **L — Load:** write the prepared data to a destination.

Example:

```text
Source database
      |
      v
EXTRACT orders
      |
      v
TRANSFORM values and validate
      |
      v
LOAD analytics database
```

## 3.1 Extract

Extraction means reading data from a source system.

Common sources include:
- PostgreSQL or MySQL databases.
- CSV and Excel files.
- JSON files.
- REST APIs.
- Application logs.
- Object storage.
- Event streams.

Example SQL:

```sql
SELECT
    order_id,
    customer_id,
    amount,
    order_status
FROM orders;
```

This query extracts selected columns from the orders table.

## 3.2 Transform

Transformation changes data into the format or meaning required by the destination.

Example source status values:

| Source value | Standard value |
|---|---|
| paid | PAID |
| Paid | PAID |
| shipped | SHIPPED |

Example Python transformation:

```python
source_status = "paid"

clean_status = source_status.strip().upper()

print(clean_status)
```

Expected output:

```text
PAID
```

We will study transformation in detail on Day 34. Today, understand where it fits in the pipeline.

## 3.3 Load

Loading writes the processed data into a destination.

Example:

```sql
INSERT INTO analytics_orders
    (order_id, customer_id, amount, order_status)
VALUES
    (1001, 201, 1500.00, 'PAID');
```

The source database and destination database may be different systems.

## 3.4 ETL versus ELT

| ETL | ELT |
|---|---|
| Extract, transform, load | Extract, load, transform |
| Transformation happens before loading to the target | Raw data is first loaded into the target platform |
| Often used when data must be cleaned before entering the destination | Common in modern cloud warehouses and data lakes |
| The transformation engine may be Python or another processing system | The warehouse/lakehouse may perform transformations using SQL or distributed compute |

Do not memorize only the letters. Understand where the transformation work takes place.

---

# 4. Understand extraction before writing code

An extraction process needs answers to several questions.

| Question | Why it matters |
|---|---|
| What is the source? | Determines how we connect and read |
| Which table or endpoint? | Defines the records to collect |
| Which columns are needed? | Avoids unnecessary data transfer |
| What is the extraction frequency? | Determines schedule and freshness |
| Full or incremental? | Determines how much data to read |
| What identifies a record? | Helps detect duplicates and updates |
| How do we know where we stopped? | Requires a watermark or checkpoint |
| What happens if the process fails? | Determines recovery design |
| How do we prove data is complete? | Requires reconciliation |

A production extraction design begins with requirements, not with blindly writing SELECT *.

---

# 5. Full extraction

## 5.1 What is full extraction?

Full extraction reads the complete required dataset from the source each time the pipeline runs.

Example:

```sql
SELECT
    order_id,
    customer_id,
    amount,
    order_status,
    updated_at
FROM orders;
```

If the source has 5,000 rows, the query may return all 5,000 rows.

## 5.2 Small beginner example

Create a Python list that represents a small source table:

```python
orders = [
    {"order_id": 1001, "customer": "Asha", "amount": 1500},
    {"order_id": 1002, "customer": "Ravi", "amount": 2800},
    {"order_id": 1003, "customer": "Meena", "amount": 900}
]

for order in orders:
    print(order)
```

Expected output:

```text
{'order_id': 1001, 'customer': 'Asha', 'amount': 1500}
{'order_id': 1002, 'customer': 'Ravi', 'amount': 2800}
{'order_id': 1003, 'customer': 'Meena', 'amount': 900}
```

The loop visits every record. That is the basic idea of a full extraction.

## 5.3 When is full extraction suitable?

Full extraction may be appropriate when:
- The source dataset is small.
- The source is a daily reference file.
- The destination is rebuilt from scratch.
- The source does not provide reliable change tracking.
- The business requires a complete snapshot.
- The extraction is inexpensive and fast enough.

## 5.4 Problems with repeated full extraction

Suppose the source has 10 million records and only 2,000 new records arrive each day.

A full extraction reads all 10 million records again, even though only 2,000 are new.

This creates unnecessary work.

Full extraction can still be correct, but it may be inefficient. Correctness and efficiency are different questions.

---

# 6. Incremental extraction

## 6.1 What is incremental extraction?

Incremental extraction reads only records that are new or have changed since a previous successful extraction.

Example:

- Yesterday's pipeline completed through 2026-11-08 23:59:59.
- Today, new or changed orders have later update timestamps.
- The pipeline reads those changed records instead of the complete table.

Basic flow:

```text
Previous successful checkpoint
             |
             v
Find new or changed source rows
             |
             v
Extract those rows
             |
             v
Validate and load
             |
             v
Save new checkpoint
```

## 6.2 Simple example

Assume the source has these rows:

| order_id | updated_at |
|---:|---|
| 1001 | 2026-11-01 09:00 |
| 1002 | 2026-11-02 10:00 |
| 1003 | 2026-11-03 11:00 |
| 1004 | 2026-11-04 12:00 |

The previous watermark is:

```text
2026-11-02 10:00
```

The next extraction should find records after that watermark.

SQL:

```sql
SELECT
    order_id,
    updated_at
FROM orders
WHERE updated_at > TIMESTAMP '2026-11-02 10:00:00'
ORDER BY updated_at, order_id;
```

Expected rows:

| order_id | updated_at |
|---:|---|
| 1003 | 2026-11-03 11:00 |
| 1004 | 2026-11-04 12:00 |

Rows 1001 and 1002 are not returned because their update timestamps are not greater than the watermark.

## 6.3 Why incremental extraction is useful

- Less data is transferred.
- Shorter processing time for many workloads.
- Lower repeated source-system load.
- More frequent refreshes become practical.
- Pipelines can process changes instead of repeatedly rebuilding everything.

Incremental extraction needs reliable change identification. If the source does not expose changes correctly, incremental logic can silently miss data.

---

# 7. What is a watermark?

A watermark is a saved value that records how far an extraction process has progressed.

It is often:
- A timestamp such as updated_at.
- A monotonically increasing ID.
- A source sequence number.
- A compound position such as (updated_at, order_id).
- A source-specific change token.

Example:

```text
Last successful watermark:
2026-11-08 18:30:00
```

The next run uses that value to decide which records to request.

## 7.1 Watermark is not the same as current time

Suppose the pipeline starts at 10:00 AM but fails at 10:05 AM.

It must not automatically save 10:05 AM as the successful watermark if records were not fully loaded.

The watermark should represent data that the pipeline has successfully processed and committed, not merely the time at which the job ran.

## 7.2 Watermark lifecycle

```text
Read previous watermark
          |
          v
Extract records after watermark
          |
          v
Validate records
          |
          v
Load records successfully
          |
          v
Commit destination transaction
          |
          v
Save new watermark
```

If the load fails, do not advance the checkpoint past unprocessed records.

---

# 8. Understand timestamp-based extraction

Many operational tables include:
- created_at: when the record was first created.
- updated_at: when the record was last modified.

Example:

| order_id | created_at | updated_at |
|---:|---|---|
| 1001 | 2026-11-01 09:00 | 2026-11-01 09:00 |
| 1002 | 2026-11-02 10:00 | 2026-11-05 14:00 |
| 1003 | 2026-11-03 11:00 | 2026-11-03 11:00 |

Order 1002 was created on November 2 but updated on November 5.

If the business needs both new and changed orders, filtering only on created_at will miss the update to order 1002.

A suitable update timestamp can identify both newly created and changed rows, provided the source maintains it correctly.

## 8.1 created_at versus updated_at

| Column | Meaning | Use |
|---|---|---|
| created_at | First creation time | Find new records |
| updated_at | Most recent modification time | Find new and changed records |

Important questions:
- Does every update change updated_at?
- Is updated_at generated by the database or application?
- Is the timestamp stored in UTC?
- Can two records have the same timestamp?
- Can a source update arrive late?
- Can a record be deleted?

A timestamp is only a reliable change marker if the source system maintains it correctly.

---

# 9. Inclusive and exclusive boundaries

This is one of the most important beginner concepts in incremental extraction.

Assume the watermark is 10:00:00.

### Exclusive condition

```sql
WHERE updated_at > TIMESTAMP '2026-11-10 10:00:00'
```

This means strictly later than 10:00:00. A row exactly at 10:00:00 is excluded.

### Inclusive condition

```sql
WHERE updated_at >= TIMESTAMP '2026-11-10 10:00:00'
```

This means 10:00:00 or later. A row exactly at 10:00:00 is included.

## 9.1 Why can > cause a missed row?

Suppose two source records have the same timestamp:

| order_id | updated_at |
|---:|---|
| 2001 | 10:00:00 |
| 2002 | 10:00:00 |

If the pipeline extracts the first row and saves 10:00:00 as its watermark, the next query using > 10:00:00 will not return the second row.

The second record can be missed.

## 9.2 Why can >= cause duplicates?

If the next run uses >= 10:00:00, it may read order 2001 again.

That can create duplicate records if the destination is not designed for safe retries.

## 9.3 Better approach: compound watermark

Use timestamp plus a unique tie-breaker such as order_id.

Suppose the last processed position is:

```text
(updated_at = 10:00:00, order_id = 2001)
```

PostgreSQL query:

```sql
SELECT
    order_id,
    updated_at
FROM orders
WHERE
    updated_at > TIMESTAMP '2026-11-10 10:00:00'
    OR (
        updated_at = TIMESTAMP '2026-11-10 10:00:00'
        AND order_id > 2001
    )
ORDER BY updated_at, order_id;
```

This retrieves rows that are later by timestamp, or have the same timestamp and a greater order ID.

This is called a compound or composite cursor.

The tie-breaker must be stable and unique for the ordering rule to work.

---

# 10. ID-based extraction

Some sources have an increasing identifier.

Example:

| event_id | event_name |
|---:|---|
| 1 | Account created |
| 2 | Order placed |
| 3 | Payment received |
| 4 | Order shipped |

If the last successfully processed ID is 3:

```sql
SELECT
    event_id,
    event_name
FROM events
WHERE event_id > 3
ORDER BY event_id;
```

Expected output:

| event_id | event_name |
|---:|---|
| 4 | Order shipped |

## 10.1 When ID-based extraction works

It can work when:
- IDs are generated in a reliably increasing order.
- New records always receive larger IDs.
- The source does not insert old IDs later.
- The pipeline only needs new inserts.

## 10.2 Important limitation

An increasing ID does not automatically detect updates to existing records.

If order 1002 changes from PENDING to PAID but keeps the same ID, a query using order_id > 1005 will never retrieve that change.

For updates, use updated_at, CDC, a change log or another source-supported change mechanism.

---

# 11. Snapshot extraction

A snapshot represents the state of a dataset at a particular point in time.

Example:

A product table is extracted every night:

```text
products_snapshot_2026_11_09.csv
products_snapshot_2026_11_10.csv
```

The files show what the product table looked like at those times.

Snapshots are useful when:
- The source provides complete daily files.
- You need historical comparison.
- The source has no reliable update timestamp.
- The dataset is small enough to copy repeatedly.

A snapshot is not automatically a change log. If a product disappears from today's snapshot, you need a rule to determine whether it was deleted, filtered out or temporarily unavailable.

---

# 12. Change Data Capture (CDC) introduction

Change Data Capture is a family of techniques for capturing inserts, updates and deletes from a source system.

A CDC event may say:
- INSERT: a new row was created.
- UPDATE: an existing row changed.
- DELETE: a row was removed.

Example event:

```json
{
  "operation": "UPDATE",
  "order_id": 1002,
  "old_status": "PENDING",
  "new_status": "PAID",
  "changed_at": "2026-11-10T10:15:00Z"
}
```

This is an illustrative event structure, not a guaranteed format from every CDC tool.

CDC can use database logs, replication slots, triggers or source-specific change feeds.

We will study the broader architecture later. For Day 30, remember that a simple updated_at watermark is not the only way to capture changes.

---

# 13. Late-arriving records

A late-arriving record is a record that becomes visible after the extraction process has already moved beyond its event time or expected arrival time.

Example:

- Pipeline watermark: November 8 at 10:00.
- A source system synchronizes an older transaction on November 9.
- The transaction's business event time is November 7.
- If the pipeline filters only on event_time > November 8, it may never retrieve it.

A reliable design must understand the source's change timestamp and arrival behavior.

## 13.1 Event time versus processing time

| Term | Meaning |
|---|---|
| Event time | When the business event actually happened |
| Processing time | When the pipeline or system processed the record |
| Updated time | When the source row was last modified |

Do not use these fields interchangeably.

A late-arriving order may have an old event_time but a recent updated_at.

## 13.2 Lookback window

A common strategy is to re-read a small overlap period.

If the saved watermark is 10:00, the next run might read from 09:45.

```sql
WHERE updated_at >= TIMESTAMP '2026-11-10 09:45:00'
ORDER BY updated_at, order_id;
```

The overlap can capture delayed visibility or timestamp edge cases.

But it will re-read some previously processed records. The destination must safely handle duplicates, usually with stable keys and a defined upsert/deduplication rule.

A lookback window does not solve every source consistency problem. Choose its size based on observed delay and business requirements.

---

# 14. Extraction frequency and source impact

Extraction can run:
- Every few minutes.
- Hourly.
- Daily.
- Weekly.
- When a source event occurs.

The correct schedule depends on the freshness requirement and source capacity.

| Requirement | Possible pattern |
|---|---|
| Daily management report | Daily batch |
| Hourly operations dashboard | Hourly incremental extraction |
| Near-real-time order tracking | Event or CDC-based architecture |
| Small reference table | Periodic full snapshot |

Do not promise real-time data if the source is only extracted once per day.

Also consider:
- Query duration.
- Source database load.
- Network capacity.
- API limits.
- Business operating hours.
- Data retention.
- Recovery time.

---

# 15. Build a beginner extraction example using Python

Before connecting to PostgreSQL, practise the extraction idea with a small in-memory dataset.

## 15.1 Source records

```python
orders = [
    {"order_id": 1001, "updated_at": "2026-11-08 09:00:00", "amount": 1500},
    {"order_id": 1002, "updated_at": "2026-11-08 10:00:00", "amount": 2800},
    {"order_id": 1003, "updated_at": "2026-11-09 11:00:00", "amount": 900},
    {"order_id": 1004, "updated_at": "2026-11-10 12:00:00", "amount": 4500}
]

watermark = "2026-11-08 10:00:00"
```

## 15.2 Extract records after the watermark

```python
new_orders = []

for order in orders:
    if order["updated_at"] > watermark:
        new_orders.append(order)

print("Extracted records:", len(new_orders))

for order in new_orders:
    print(order)
```

Expected output:

```text
Extracted records: 2
{'order_id': 1003, 'updated_at': '2026-11-09 11:00:00', 'amount': 900}
{'order_id': 1004, 'updated_at': '2026-11-10 12:00:00', 'amount': 4500}
```

### Understand the code line by line

- new_orders = [] creates an empty list to store extracted rows.
- for order in orders visits each source record.
- order["updated_at"] reads the update timestamp.
- > watermark checks whether the record is later than the saved watermark.
- append(order) adds qualifying records to the extraction result.
- len(new_orders) counts the extracted records.

This example uses timestamps in a consistent ISO-like text format. Lexicographic comparison works for this fixed format, but production systems should use actual datetime objects or database timestamp comparisons with clear timezone rules.

## 15.3 Inspect each extracted record

```python
for order in new_orders:
    print(
        "Order ID:", order["order_id"],
        "| Amount:", order["amount"]
    )
```

Expected output:

```text
Order ID: 1003 | Amount: 900
Order ID: 1004 | Amount: 4500
```

The loop reads values from each dictionary using its key.

---

# 16. Work with actual timestamps in Python

For real Python code, use datetime values rather than comparing arbitrary date strings.

```python
from datetime import datetime

watermark = datetime.fromisoformat("2026-11-08 10:00:00")

order_time = datetime.fromisoformat("2026-11-09 11:00:00")

if order_time > watermark:
    print("This order is newer than the watermark")
```

Expected output:

```text
This order is newer than the watermark
```

### Important timezone lesson

A timestamp with no timezone is called a naive datetime in Python. A timezone-aware datetime includes timezone information.

For production extraction:
- Agree whether source timestamps are UTC.
- Store and compare timestamps consistently.
- Avoid comparing naive and timezone-aware datetimes.
- Be careful with daylight-saving transitions in systems that use local time.
- Use the source database's documented timestamp semantics.

PostgreSQL TIMESTAMPTZ represents an instant and displays it according to the session timezone. MySQL timestamp handling differs and must be configured and understood explicitly.

---

# 17. Connect the extraction query to PostgreSQL

We will now connect the concept to the database skills from Day 29.

Assume a source table named source_orders:

| order_id | customer_id | amount | order_status | updated_at |
|---:|---:|---:|---|---|
| 1001 | 201 | 1500.00 | PAID | 2026-11-08 09:00 |
| 1002 | 202 | 2800.00 | SHIPPED | 2026-11-08 10:00 |
| 1003 | 203 | 900.00 | PENDING | 2026-11-09 11:00 |
| 1004 | 204 | 4500.00 | PAID | 2026-11-10 12:00 |

The values above are the lesson's example data. If you use a different database, insert the same rows or adapt the query to your own schema.

## 17.1 Create the practice source table

Run in PostgreSQL:

```sql
CREATE TABLE IF NOT EXISTS source_orders (
    order_id INTEGER PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    amount NUMERIC(12, 2) NOT NULL,
    order_status VARCHAR(30) NOT NULL,
    updated_at TIMESTAMP NOT NULL
);
```

This creates a table if it does not already exist.

## 17.2 Insert the sample rows

For a repeatable classroom setup, use an upsert:

```sql
INSERT INTO source_orders
    (order_id, customer_id, amount, order_status, updated_at)
VALUES
    (1001, 201, 1500.00, 'PAID', '2026-11-08 09:00:00'),
    (1002, 202, 2800.00, 'SHIPPED', '2026-11-08 10:00:00'),
    (1003, 203, 900.00, 'PENDING', '2026-11-09 11:00:00'),
    (1004, 204, 4500.00, 'PAID', '2026-11-10 12:00:00')
ON CONFLICT (order_id)
DO UPDATE SET
    customer_id = EXCLUDED.customer_id,
    amount = EXCLUDED.amount,
    order_status = EXCLUDED.order_status,
    updated_at = EXCLUDED.updated_at;
```

This example resets the four sample records to the stated values each time it runs. Do not use this setup statement against real production data.

## 17.3 Run the full extraction query

```sql
SELECT
    order_id,
    customer_id,
    amount,
    order_status,
    updated_at
FROM source_orders
ORDER BY order_id;
```

Expected: four rows in order ID order.

## 17.4 Run the incremental extraction query

```sql
SELECT
    order_id,
    customer_id,
    amount,
    order_status,
    updated_at
FROM source_orders
WHERE updated_at > TIMESTAMP '2026-11-08 10:00:00'
ORDER BY updated_at, order_id;
```

Expected:

| order_id | customer_id | amount | order_status | updated_at |
|---:|---:|---:|---|---|
| 1003 | 203 | 900.00 | PENDING | 2026-11-09 11:00 |
| 1004 | 204 | 4500.00 | PAID | 2026-11-10 12:00 |

### Why are only two rows returned?

- Order 1001 is earlier than the watermark.
- Order 1002 equals the watermark and > excludes equality.
- Orders 1003 and 1004 are later than the watermark.

This is the exact meaning of the filter.

---

# 18. Execute the extraction from Python with Psycopg 3

This exercise assumes you completed Day 29 and have Psycopg 3 installed.

Create a practice project:

```text
day30_incremental_extraction/
|
|-- .env
|-- .gitignore
|-- 01_full_extract.py
|-- 02_incremental_extract.py
|-- 03_extract_and_report.py
```

Use the same secure environment-variable setup taught on Day 29. Do not commit .env or passwords.

## 18.1 Full extraction using Python

Create 01_full_extract.py:

```python
import os
import psycopg
from dotenv import load_dotenv

load_dotenv()

with psycopg.connect(
    host=os.getenv("DB_HOST"),
    port=os.getenv("DB_PORT"),
    dbname=os.getenv("DB_NAME"),
    user=os.getenv("DB_USER"),
    password=os.getenv("DB_PASSWORD")
) as connection:

    with connection.cursor() as cursor:
        cursor.execute("""
            SELECT
                order_id,
                customer_id,
                amount,
                order_status,
                updated_at
            FROM source_orders
            ORDER BY order_id
        """)

        rows = cursor.fetchall()

        print("Total extracted:", len(rows))

        for row in rows:
            print(row)
```

Run:

```bash
python 01_full_extract.py
```

Expected: four rows and a count of four.

## 18.2 Incremental extraction using a fixed watermark

Create 02_incremental_extract.py:

```python
import os
import psycopg
from dotenv import load_dotenv

load_dotenv()

watermark = "2026-11-08 10:00:00"

with psycopg.connect(
    host=os.getenv("DB_HOST"),
    port=os.getenv("DB_PORT"),
    dbname=os.getenv("DB_NAME"),
    user=os.getenv("DB_USER"),
    password=os.getenv("DB_PASSWORD")
) as connection:

    with connection.cursor() as cursor:
        cursor.execute(
            """
            SELECT
                order_id,
                customer_id,
                amount,
                order_status,
                updated_at
            FROM source_orders
            WHERE updated_at > %s
            ORDER BY updated_at, order_id
            """,
            (watermark,)
        )

        rows = cursor.fetchall()

        print("Watermark:", watermark)
        print("Incremental records:", len(rows))

        for row in rows:
            print(row)
```

Expected:
- Watermark shown as 2026-11-08 10:00:00.
- Two incremental records.
- Orders 1003 and 1004 returned.

The parameter value is supplied separately from SQL. Psycopg adapts the value for the database. In a production implementation, use a proper timezone-aware datetime or database timestamp type consistent with the source column.

---

# 19. A real checkpoint table

A pipeline must remember its previous successful position.

One common design is a control table.

PostgreSQL example:

```sql
CREATE TABLE IF NOT EXISTS etl_checkpoints (
    pipeline_name VARCHAR(100) PRIMARY KEY,
    last_updated_at TIMESTAMP,
    last_order_id INTEGER,
    updated_on TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

This table stores the extraction position for each pipeline.

Example row:

| pipeline_name | last_updated_at | last_order_id |
|---|---|---:|
| retail_orders | 2026-11-08 10:00:00 | 1002 |

The combination of timestamp and ID forms a compound checkpoint.

## 19.1 Insert the first checkpoint

```sql
INSERT INTO etl_checkpoints
    (pipeline_name, last_updated_at, last_order_id)
VALUES
    ('retail_orders', '2026-11-08 10:00:00', 1002)
ON CONFLICT (pipeline_name)
DO UPDATE SET
    last_updated_at = EXCLUDED.last_updated_at,
    last_order_id = EXCLUDED.last_order_id,
    updated_on = CURRENT_TIMESTAMP;
```

This initializes or updates the checkpoint for the named pipeline.

## 19.2 Read the checkpoint

```sql
SELECT
    pipeline_name,
    last_updated_at,
    last_order_id
FROM etl_checkpoints
WHERE pipeline_name = 'retail_orders';
```

The pipeline reads this row before extracting the next batch.

## 19.3 Do not update the checkpoint too early

Incorrect sequence:

```text
Read source
   |
   v
Update checkpoint
   |
   v
Load destination
   |
   v
Load fails
```

The checkpoint now claims that records were processed, even though the destination load failed. A later run may skip them.

Safer sequence:

```text
Read checkpoint
   |
   v
Extract source rows
   |
   v
Validate and load destination
   |
   v
Commit destination changes
   |
   v
Persist the new checkpoint
```

If the destination data and checkpoint are in the same database, they may be updated in the same transaction. If they are in separate systems, exactly-once behavior is harder; design for safe retries and reconciliation rather than assuming a distributed transaction exists.

---

# 20. Build the mini-project: Incremental Extraction Pipeline

## 20.1 Business requirement

The retail company wants to extract new or updated order records from PostgreSQL without scanning the entire source table on every run.

The pipeline must:
- Read a previous checkpoint.
- Extract records after that position.
- Use deterministic ordering.
- Print the extracted batch.
- Calculate the new high-water position.
- Explain when it is safe to save the checkpoint.

For this first mini-project, we will demonstrate extraction and checkpoint calculation. The next lessons will expand this into full ingestion and loading.

## 20.2 Create the checkpoint

Run this once in PostgreSQL:

```sql
INSERT INTO etl_checkpoints
    (pipeline_name, last_updated_at, last_order_id)
VALUES
    ('retail_orders', '2026-11-08 10:00:00', 1002)
ON CONFLICT (pipeline_name)
DO UPDATE SET
    last_updated_at = EXCLUDED.last_updated_at,
    last_order_id = EXCLUDED.last_order_id;
```

This sets the practice starting point.

## 20.3 Extraction query with a compound watermark

```sql
SELECT
    order_id,
    customer_id,
    amount,
    order_status,
    updated_at
FROM source_orders
WHERE
    updated_at > %s
    OR (
        updated_at = %s
        AND order_id > %s
    )
ORDER BY updated_at, order_id;
```

The three parameters are:
1. Previous timestamp.
2. The same previous timestamp for equality comparison.
3. Previous order ID.

This query supports multiple rows with the same update timestamp, as long as order_id is unique and the ordering is stable.

## 20.4 Complete beginner Python extraction script

Create 03_extract_and_report.py:

```python
import os
import psycopg
from dotenv import load_dotenv

load_dotenv()

pipeline_name = "retail_orders"

with psycopg.connect(
    host=os.getenv("DB_HOST"),
    port=os.getenv("DB_PORT"),
    dbname=os.getenv("DB_NAME"),
    user=os.getenv("DB_USER"),
    password=os.getenv("DB_PASSWORD")
) as connection:

    with connection.cursor() as cursor:

        cursor.execute(
            """
            SELECT last_updated_at, last_order_id
            FROM etl_checkpoints
            WHERE pipeline_name = %s
            """,
            (pipeline_name,)
        )

        checkpoint = cursor.fetchone()

        if checkpoint is None:
            raise ValueError("Checkpoint is missing for this pipeline")

        last_updated_at = checkpoint[0]
        last_order_id = checkpoint[1]

        print("Previous timestamp:", last_updated_at)
        print("Previous order ID:", last_order_id)

        cursor.execute(
            """
            SELECT
                order_id,
                customer_id,
                amount,
                order_status,
                updated_at
            FROM source_orders
            WHERE
                updated_at > %s
                OR (
                    updated_at = %s
                    AND order_id > %s
                )
            ORDER BY updated_at, order_id
            """,
            (last_updated_at, last_updated_at, last_order_id)
        )

        extracted_rows = cursor.fetchall()

        print("Extracted row count:", len(extracted_rows))

        for row in extracted_rows:
            print(row)

        if len(extracted_rows) > 0:
            latest_row = extracted_rows[-1]

            new_updated_at = latest_row[4]
            new_order_id = latest_row[0]

            print("Candidate next timestamp:", new_updated_at)
            print("Candidate next order ID:", new_order_id)
        else:
            print("No new records found")
```

Expected with the sample data:

- Previous checkpoint: November 8 at 10:00 and order ID 1002.
- Extracted row count: 2.
- Rows 1003 and 1004.
- Candidate next checkpoint: November 10 at 12:00 and order ID 1004.

### Understand the script carefully

**Part A — Read the checkpoint**

The SELECT reads the saved timestamp and ID for the pipeline. fetchone returns one row because pipeline_name is a primary key.

**Part B — Check for a missing checkpoint**

If checkpoint is None, there is no saved starting position. The script stops instead of silently assuming a value.

**Part C — Extract records**

The WHERE clause compares the source position with the saved position. The ORDER BY clause ensures rows are returned in the same order used by the compound checkpoint.

**Part D — Calculate the candidate watermark**

The last row in the ordered result has the highest extracted position. We call it a candidate because it must not be persisted until the corresponding data has been safely loaded.

### Important limitation of this teaching script

This script demonstrates checkpoint reading and extraction. It does not yet load rows into a separate destination table or advance the checkpoint. That is intentional: advancing a checkpoint without a successful destination write would teach an unsafe pipeline pattern.

---

# 21. Common extraction mistakes

## Mistake 1: Always using SELECT *

Why it is a problem:
- Reads unnecessary columns.
- Transfers more data.
- Can break downstream code when source schema changes.

Better approach: select only the columns required by the pipeline.

## Mistake 2: Using created_at when updates must be captured

A changed order may keep its original created_at. Use a trustworthy updated_at, CDC feed or source change log when updates matter.

## Mistake 3: Saving the current clock time as the watermark

The job's start time is not proof that all earlier source records were processed. Save a position derived from successfully processed data.

## Mistake 4: Updating the checkpoint before loading

A failed load can leave the checkpoint ahead of the destination. Advance only after successful processing and commit.

## Mistake 5: Using timestamp alone when timestamps can tie

Two records can share the same timestamp. Add a unique tie-breaker or use a source-supported change sequence.

## Mistake 6: Ignoring late-arriving data

A record may become visible after the pipeline has passed its business event time. Understand the source's update timestamp and consider overlap or CDC.

## Mistake 7: Assuming increasing IDs capture updates

An ID-based query normally finds new IDs, not changes to existing rows.

## Mistake 8: Ignoring deletes

A row that is deleted from the source will not appear in a normal SELECT of current rows. Use CDC, tombstones, soft-delete flags or snapshot comparison when deletes matter.

## Mistake 9: Not using deterministic ordering

If you calculate a checkpoint from the last row, the query must sort by the same fields that define the checkpoint.

## Mistake 10: Mixing timezone conventions

A local timestamp and a UTC timestamp can represent different instants. Standardize timezone handling.

---

# 22. Extraction troubleshooting

| Symptom | Possible cause | What to check |
|---|---|---|
| No rows returned | Watermark is too recent | Inspect checkpoint and source timestamps |
| A row is missing | Timestamp tie or late arrival | Check boundary rule and source update behavior |
| Same rows appear repeatedly | Inclusive boundary or lookback | Ensure destination is retry-safe |
| Updated record is missing | Filter uses created_at | Use updated_at or CDC |
| Duplicate rows in target | No stable key or conflict policy | Define primary/business key and upsert behavior |
| Job is slow | Full scan or unnecessary columns | Inspect query plan and source indexes |
| Checkpoint advanced but target is incomplete | Checkpoint saved too early | Couple destination commit and checkpoint strategy |
| Deleted row remains in target | Extraction does not capture deletes | Use delete tracking or reconciliation |

---

# 23. Source-system impact and extraction performance

Extraction is not free. Queries consume resources on the source.

Good practices:
1. Select only required columns.
2. Filter on an appropriate indexed change column when possible.
3. Use deterministic ordering.
4. Read in manageable batches for large datasets.
5. Avoid long-running queries during source peak periods when possible.
6. Coordinate with the source-system owner.
7. Use replicas or CDC infrastructure where appropriate.
8. Inspect query plans before running expensive production queries.
9. Record extraction duration and row counts.
10. Avoid changing source data as part of a read-only extraction job.

An index on updated_at may help a timestamp filter, but index design depends on table size, selectivity, write load and query pattern. Indexes also have storage and write costs.

---

# 24. Exercises

Try each question before reading the answer.

## Exercise 1 — Full or incremental?

A source table contains 50,000 customer records. The pipeline needs a complete copy every Sunday, and the source is small.

**Question:** Is full extraction reasonable?

**Answer:** Yes, if the source and destination can handle it and a complete weekly snapshot is required. Incremental extraction is not automatically better for every workload.

## Exercise 2 — Find new records

Watermark = 2026-11-01 12:00:00.

| ID | updated_at |
|---:|---|
| 1 | 2026-11-01 10:00 |
| 2 | 2026-11-01 12:00 |
| 3 | 2026-11-01 13:00 |
| 4 | 2026-11-02 09:00 |

Using > watermark, which records are returned?

**Answer:** IDs 3 and 4.

## Exercise 3 — Detect the update

Order 1005 was created on November 1 and updated on November 5.

Which column should an incremental query use if it must capture changes?

**Answer:** A reliable updated_at column, or a CDC/change-log mechanism.

## Exercise 4 — Find the checkpoint bug

The job saves its new watermark before inserting records into the destination. The insert fails.

What is the risk?

**Answer:** The next run may start after records that were never loaded, causing data loss.

## Exercise 5 — Same timestamp

Two records have timestamp 10:00:00 and IDs 51 and 52. The previous checkpoint is (10:00:00, 51).

Which record should the next compound-watermark query return?

**Answer:** ID 52.

## Exercise 6 — Late-arriving record

A record has event_time November 2 but updated_at November 6. The pipeline runs November 6.

Which timestamp is useful for detecting the newly visible update?

**Answer:** updated_at, assuming the source updates it correctly.

---

# 25. Day 30 mini-project submission

**Project:** Incremental Extraction Pipeline

## Required deliverables

1. A source table or clearly documented source dataset.
2. A full extraction query.
3. An incremental extraction query.
4. A checkpoint table.
5. A Python script that reads the checkpoint.
6. A deterministic extraction query.
7. Extracted row count and sample output.
8. Candidate next watermark.
9. Explanation of when checkpoint persistence is safe.
10. A short troubleshooting note.

## Validation checklist

- [ ] Full extraction returns the complete sample dataset.
- [ ] Incremental extraction returns only records after the checkpoint.
- [ ] The equality boundary is understood.
- [ ] The query has deterministic ORDER BY.
- [ ] A unique tie-breaker is included.
- [ ] The checkpoint is read from a control table.
- [ ] The script does not advance the checkpoint before a successful load.
- [ ] Late-arriving records are discussed.
- [ ] Updates and deletes are considered separately.
- [ ] The extracted count is checked.

## What students should explain to a mentor

1. Why did you choose incremental extraction?
2. What does the watermark represent?
3. Why is updated_at different from created_at?
4. Why did you use order_id as a tie-breaker?
5. What happens if two rows have the same timestamp?
6. Why must the checkpoint be saved after successful loading?
7. How could a late-arriving record be missed?
8. How would you capture deletes?
9. How would you reduce source database load?
10. How would you make the pipeline safe to retry?

---

# 26. Day 30 revision

| Concept | Remember |
|---|---|
| ETL | Extract, Transform, Load |
| Extraction | Reading data from a source |
| Full extraction | Read the complete required dataset |
| Incremental extraction | Read new or changed records |
| Watermark | Saved extraction progress position |
| created_at | When a row was created |
| updated_at | When a row was last modified |
| Compound watermark | Timestamp plus a stable tie-breaker |
| Lookback window | Re-read a small overlap period |
| CDC | Capture inserts, updates and deletes from source changes |
| Checkpoint | Persisted progress used by the next run |
| Reconciliation | Check that extracted and loaded data agree |

**Today's most important rule:** Never move an extraction checkpoint beyond data that the pipeline has successfully processed. A pipeline must be designed for failure and safe retry, not just for its successful first run.

**Next day:** Day 31 — File-Based Data Ingestion. We will take the extraction concepts and build a practical system that discovers and processes multiple files from a directory.
