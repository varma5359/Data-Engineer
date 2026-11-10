# Day 29: Python and Database Integration

**Pathway:** TechPhile.ai — Data Engineer Career Pathway  
**Phase:** 3 — SQL and Database Engineering  
**Audience:** Freshers, college students and junior Data Engineer candidates  
**Level:** Beginner to junior | **Class time:** 2–3 hours  
**Approved syllabus:** Python database drivers, connection handling, bulk inserts, batch operations, transaction handling, database loading  
**Mini-project:** Database Loader  
**Primary database:** PostgreSQL; MySQL differences are identified where relevant.

---

## 1. How to study

Read one concept, predict what the query or program will do, execute the small example, compare the result and explain it aloud. Do not run the entire lesson as one large script.

## 2. Prerequisite and learning outcomes

Use the TechStore training database from Day 20: customers, products, orders and order_items. Confirm you are connected to the correct practice database. If sample data differs, inspect it before comparing outputs. Never experiment on production data.

By the end, you should understand each item in today's syllabus, execute the main examples, explain the output, recognize common mistakes and complete **Database Loader**.

## 3. Why Python and database?

Python extracts files/APIs and transforms records. A relational database stores structured data and enforces constraints. Typical flow: source → Python extract → validate → staging table → verify → downstream pipeline.

## 4. Prepare environment

Need Python, PostgreSQL access, a role allowed to connect/insert, and Psycopg 3.

Windows CMD:
```bash
python --version
python -m pip install "psycopg[binary]"
python -m pip show psycopg
```
Binary extra is convenient for training; production dependency policy may differ.

Create dedicated staging table:
```sql
CREATE TABLE IF NOT EXISTS customer_load_stage(
 source_customer_id INTEGER PRIMARY KEY,
 customer_name VARCHAR(100) NOT NULL,
 email VARCHAR(150) NOT NULL,
 city VARCHAR(60),
 loaded_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
SELECT * FROM customer_load_stage;
```
Expected initially zero rows. Keep it separate from customers.

## 5. Connection settings safely

CMD session variables (replace placeholders privately):
```bat
set PGHOST=localhost
set PGPORT=5432
set PGDATABASE=techstore_training
set PGUSER=your_training_user
set PGPASSWORD=your_training_password
```
Do not commit credentials or print them. Production should use a secret manager or managed identity.

## 6. First connection

connect_db.py:
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
print("Database connection successful")
connection.close()
```
Run `python connect_db.py`. Expected: Database connection successful. os.environ reads environment variables; connect opens session; close releases it. If error, check host, port, database, credentials, server and network.

## 7. Read rows with cursor

```python
import os
import psycopg

connection = psycopg.connect(
    host=os.environ["PGHOST"], port=os.environ["PGPORT"],
    dbname=os.environ["PGDATABASE"], user=os.environ["PGUSER"],
    password=os.environ["PGPASSWORD"]
)
cursor = connection.cursor()
cursor.execute("SELECT customer_id,customer_name,city FROM customers ORDER BY customer_id")
rows = cursor.fetchall()
for row in rows:
    print(row)
cursor.close()
connection.close()
```
Expected:
```text
(1, 'Asha', 'Hyderabad')
(2, 'Ravi', 'Chennai')
(3, 'Meena', 'Bengaluru')
```
cursor executes SQL; fetchall retrieves all rows as tuples. Large extracts should use fetchmany or cursor iteration to avoid memory spikes.

## 8. Parameterized SQL

Never concatenate user input into SQL. It risks SQL injection and quoting errors.

```python
customer_id = 1
cursor.execute(
    "SELECT customer_id,customer_name FROM customers WHERE customer_id=%s",
    (customer_id,)
)
print(cursor.fetchone())
```
Expected (1,'Asha'). Psycopg uses %s placeholders; values are sent separately. One-value tuple requires trailing comma. Other drivers may use different placeholders.

## 9. Insert and commit

```python
new_customer = (4,"Kiran","kiran@example.com","Pune")
cursor.execute(
    """INSERT INTO customer_load_stage
       (source_customer_id,customer_name,email,city)
       VALUES (%s,%s,%s,%s)""",
    new_customer
)
connection.commit()
print("One customer row inserted")
```
Expected success message if execute and commit both succeed. Verify with SELECT from customer_load_stage. Commit makes changes durable/visible according to database transaction guarantees. Print success only after commit.

## 10. Batch insert

```python
customer_rows = [
 (5,"Anu","anu@example.com","Delhi"),
 (6,"Dev","dev@example.com","Mumbai"),
 (7,"Sara","sara@example.com","Pune")
]
cursor.executemany(
 """INSERT INTO customer_load_stage
    (source_customer_id,customer_name,email,city)
    VALUES (%s,%s,%s,%s)""",
 customer_rows
)
connection.commit()
print("Batch inserted:",len(customer_rows))
```
Expected Batch inserted: 3. Count is 4 after Kiran plus three rows. executemany is convenient but optimization depends on driver; benchmark COPY/bulk loading for large volumes.

## 11. Transaction error handling

```python
try:
    cursor.execute(
        """INSERT INTO customer_load_stage
           (source_customer_id,customer_name,email,city)
           VALUES (%s,%s,%s,%s)""",
        (8,"Nila","nila@example.com","Kochi")
    )
    connection.commit()
    print("Transaction committed")
except Exception as error:
    connection.rollback()
    print("Transaction rolled back")
    print("Error type:",type(error).__name__)
```
Expected first run: Transaction committed. Duplicate key: exception and rollback. In production catch expected driver exceptions, log safe context, classify retryable/permanent errors; don't hide every error and continue.

## 12. Context managers

Psycopg connection context commits on successful exit, rolls back on exception and closes connection when leaving the block. Cursor context closes cursor.

```python
import os
import psycopg

with psycopg.connect(
 host=os.environ["PGHOST"],port=os.environ["PGPORT"],
 dbname=os.environ["PGDATABASE"],user=os.environ["PGUSER"],
 password=os.environ["PGPASSWORD"]
) as connection:
    with connection.cursor() as cursor:
        cursor.execute("SELECT COUNT(*) FROM customer_load_stage")
        result=cursor.fetchone()
        print("Rows in staging table:",result[0])
```
Expected count depends on completed inserts: 4 after Kiran/Anu/Dev/Sara, 5 after Nila.

## 13. Idempotency and upsert

A retry may occur after commit but before client receives confirmation. Primary key prevents silent duplicate ID; pipeline must define conflict policy.

```sql
INSERT INTO customer_load_stage
(source_customer_id,customer_name,email,city)
VALUES (4,'Kiran Updated','kiran@example.com','Pune')
ON CONFLICT (source_customer_id)
DO UPDATE SET customer_name=EXCLUDED.customer_name,
 email=EXCLUDED.email,city=EXCLUDED.city;
```
EXCLUDED means proposed insert row. Upsert only with trusted source key and update policy; stale source data may overwrite newer data.

## 14. Batch size

Read bounded batches, validate, load, commit at agreed boundary, record batch ID/counts, repeat. Small batches use less memory but more round trips; large batches use more memory and hold locks longer. One huge transaction can create large rollback work. Tune by measurement.

## 15. Mini-project — Database Loader

Write a simple Python loader that reads a small in-memory list, connects via environment variables, uses parameterized SQL, inserts batches, commits success, rolls back failure and verifies target count. Use staging table and dedicated source IDs.

Sample:
```python
customer_rows = [
 (101,"Isha","isha@example.com","Hyderabad"),
 (102,"Arun","arun@example.com","Chennai"),
 (103,"Maya","maya@example.com","Bengaluru")
]
```
Deliverables: Python script, requirements file, SQL setup, output, verification query, transaction/retry explanation.

## 16. Common mistakes and self-check

Avoid hard-coded secrets, string-built SQL, missing tuple comma, forgetting commit/rollback, success before commit, fetchall on huge data, connection per row, blind retries, swallowing exceptions.

Explain driver, cursor, execute, fetchone/fetchall, parameterized SQL, commit, rollback, executemany, COPY, idempotency, upsert, batches, context managers and row-count reconciliation.

## 99. Next lesson

Phase 3 is complete. Continue to Phase 4 — Data Processing and Pipelines.