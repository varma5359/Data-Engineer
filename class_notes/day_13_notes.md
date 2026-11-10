# Day 13: Python Database Connectivity

**Phase:** 2 — Python for Data Engineering  
**Mini-project:** Database Record Extractor  
**Learning time:** 2–3 hours

## 1. Learning outcomes
- Explain how Python connects to relational databases.
- Understand connection, cursor and transaction.
- Execute parameterized SQL safely.
- Fetch query results and handle database errors.
- Manage connections and bulk extraction responsibly.

## 2. Why connect Python to a database?
Python pipelines often need to read operational records, load transformed data, execute SQL checks or coordinate database tasks. The database remains responsible for storing and querying relational data; Python manages the surrounding workflow.

## 3. Connection concepts

| Term | Meaning |
|---|---|
| Driver | Library that speaks a database's protocol |
| Connection | Session between application and database |
| Cursor | Object used to execute statements and fetch results |
| Transaction | Group of operations committed or rolled back together |
| Commit | Persist transaction changes |
| Rollback | Undo uncommitted changes |

Credentials, host, database name and permissions must come from approved configuration. Never embed production passwords in source files.

## 4. Install a driver
For PostgreSQL, a common driver is psycopg:

```bash
python -m pip install "psycopg[binary]"
```

For MySQL, use an approved connector such as mysql-connector-python. Driver APIs differ; read the documentation for the selected version.

## 5. Connect to PostgreSQL
```python
import os
import psycopg

connection = psycopg.connect(
    host=os.environ["DB_HOST"],
    dbname=os.environ["DB_NAME"],
    user=os.environ["DB_USER"],
    password=os.environ["DB_PASSWORD"],
    port=os.environ.get("DB_PORT", "5432")
)

print("Database connection established")
```

The environment variables must be configured before execution. Do not print the password or full connection string.

## 6. Execute a SELECT
```python
with connection.cursor() as cursor:
    cursor.execute("SELECT order_id, status FROM orders LIMIT 5")
    rows = cursor.fetchall()

for row in rows:
    print(row)
```

A cursor executes SQL and retrieves result rows. `fetchall()` loads the selected result into memory, so limit or page large result sets.

## 7. Fetch one or many rows
```python
with connection.cursor() as cursor:
    cursor.execute("SELECT order_id FROM orders ORDER BY order_id")
    first_row = cursor.fetchone()
    print(first_row)
```

Fetch in bounded batches:

```python
with connection.cursor() as cursor:
    cursor.execute("SELECT order_id, amount FROM orders ORDER BY order_id")

    while True:
        batch = cursor.fetchmany(1000)

        if not batch:
            break

        print("Batch records:", len(batch))
```

For very large tables, also consider server-side cursors, keyset pagination, query plans and source load.

## 8. Parameterized queries
Never build SQL by concatenating untrusted values.

Unsafe pattern:

```python
# Do not do this with untrusted input:
# query = "SELECT * FROM orders WHERE customer_id = " + customer_id
```

Safe parameterized pattern (psycopg):

```python
customer_id = 1001

with connection.cursor() as cursor:
    cursor.execute(
        "SELECT order_id, amount FROM orders WHERE customer_id = %s",
        (customer_id,)
    )
    rows = cursor.fetchall()

print(rows)
```

The placeholder syntax depends on the driver. Parameterization protects values; it does not allow table or column identifiers to be passed as ordinary value parameters.

## 9. Insert and commit
```python
with connection.cursor() as cursor:
    cursor.execute(
        "INSERT INTO pipeline_audit (run_id, status) VALUES (%s, %s)",
        ("run_001", "STARTED")
    )

connection.commit()
```

A transaction may be rolled back if an error occurs before commit. Understand the driver's autocommit behavior.

## 10. Rollback on failure
```python
try:
    with connection.cursor() as cursor:
        cursor.execute(
            "UPDATE orders SET status = %s WHERE order_id = %s",
            ("PAID", 501)
        )
    connection.commit()

except Exception:
    connection.rollback()
    raise
```

Re-raising preserves failure visibility. A production implementation should catch expected database exceptions at the correct boundary and log safe context.

## 11. Close connections
```python
connection.close()
print("Connection closed")
```

Use context managers or a connection pool where appropriate. Do not create a new database connection for every row.

## 12. Bulk operations
Inserting one row at a time creates repeated network and transaction overhead. Use driver-supported batch operations or staging tables for large loads. Choose batch size based on row width, database limits and memory.

A common pattern:
1. Extract a bounded source batch.
2. Validate and transform.
3. Bulk load into a staging table.
4. Merge/upsert into the target.
5. Commit and record metrics.
6. Advance checkpoint after safe commit.

## 13. Mini-project: Database Record Extractor
Prepare a training database with an `orders` table. Example query:

```sql
SELECT order_id, customer_id, order_date, amount, status
FROM orders
WHERE updated_at >= %s
ORDER BY updated_at, order_id;
```

Python extraction:

```python
from datetime import datetime

watermark = datetime(2026, 1, 1)

with connection.cursor() as cursor:
    cursor.execute(
        """
        SELECT order_id, customer_id, amount, status
        FROM orders
        WHERE updated_at >= %s
        ORDER BY updated_at, order_id
        """,
        (watermark,)
    )

    rows = cursor.fetchall()

print("Extracted:", len(rows))
print(rows[:5])
```

For production, use a stable watermark strategy, account for equal timestamps and late updates, and persist the last successfully committed position.

## 14. Security and operational considerations
- Use a dedicated service account with least privilege.
- Store credentials in approved secret storage.
- Encrypt network connections where required.
- Set connection and statement timeouts.
- Avoid SELECT * for production extraction.
- Limit query scope and avoid overwhelming OLTP systems.
- Record query/run metadata without sensitive values.
- Use connection pooling for repeated workloads where appropriate.

## 15. Common mistakes
| Mistake | Better practice |
|---|---|
| SQL string concatenation | Parameterized queries |
| Never committing writes | Understand transaction boundaries |
| Committing each row | Use controlled batches |
| Fetching huge results at once | fetchmany/keyset/server-side cursor |
| Leaving connections open | Context managers and cleanup |
| Using admin credentials | Dedicated least-privilege account |
| Advancing watermark too early | Commit target then checkpoint |

## 16. Day 13 revision
- A driver connects Python to a database.
- Cursor executes SQL and fetches rows.
- Parameterized queries separate values from SQL text.
- Transactions provide commit/rollback boundaries.
- Batch extraction and loading control resource usage.
- Connection lifecycle, security and checkpoints are part of pipeline correctness.
