# Day 13 — Database Connections and SQL Execution from Python
**Phase 2: Python for Data Engineering**

## 1. Why connect Python to databases?
Python can automate extraction/loading while databases execute SQL and enforce constraints. Use a supported driver (e.g., psycopg for PostgreSQL, mysql-connector-python for MySQL, or SQLAlchemy).

## 2. Connection safety
- Use environment/secret manager for credentials.
- Use least-privilege service account.
- Set connection/query timeouts where supported.
- Close connections/cursors; use context managers when available.
- Never concatenate user values into SQL strings.

## 3. Parameterized query (illustrative SQLite)
```python
import sqlite3
connection = sqlite3.connect("sales.db")
cursor = connection.cursor()
cursor.execute("SELECT order_id, amount FROM orders WHERE amount > ?", (500,))
rows = cursor.fetchall()
print(rows)
connection.close()
```
The `?` placeholder binds a value safely. Placeholder syntax varies by driver.

## 4. Insert and transaction
```python
connection = sqlite3.connect("sales.db")
cursor = connection.cursor()
cursor.execute("INSERT INTO orders (order_id, amount) VALUES (?, ?)", (101, 750))
connection.commit()
connection.close()
```
Commit makes the transaction durable; if a multi-step operation fails, rollback may be needed.

## 5. Bulk operations
For many rows, use driver-supported executemany/bulk loading rather than one network round-trip per record. Batch size should balance throughput, memory, and transaction duration.

## 6. Mini-project — Database Record Extractor
Connect to a local sample database, run a parameterized query, fetch records in batches, and export results. Track query/run metadata and close resources.

## Common mistakes
- SQL injection through string concatenation.
- Credentials in code.
- Fetching millions of rows at once.
- Forgetting commit/rollback.
- Leaving connection open.
- Assuming one driver's placeholders work in another.

## Revision
Python connects through drivers; parameterized SQL protects values; transactions control commit/rollback; bulk operations improve throughput.