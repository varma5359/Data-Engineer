# Day 13: Python Database Connectivity

**Phase:** 2 — Python for Data Engineering  
**Mini-project:** Database Record Extractor  
**Learning time:** 2–3 hours

## 1. Learning outcomes

- Explain how Python communicates with relational databases.
- Understand database drivers, connections, cursors and transactions.
- Create a local SQLite database for practice.
- Create tables and insert records from Python.
- Execute SELECT queries and retrieve results.
- Use parameterized queries safely.
- Update and delete records.
- Commit or roll back transactions.
- Extract records in batches and manage connections.

## 2. Why connect Python to a database?

A data engineer often needs to extract records from operational systems, validate data, load transformed results or automate SQL tasks. Python coordinates the workflow while the database stores and queries the data.

Example:

```text
Python script -> Database connection -> SQL query -> Result rows
             -> Validation / transformation -> Output file
```

## 3. Understand the main database terms

| Term | Meaning |
|---|---|
| Database driver | Library that allows Python to communicate with a database |
| Connection | Active session between Python and the database |
| Cursor | Object used to execute SQL statements and fetch results |
| SQL statement | Command such as SELECT, INSERT or UPDATE |
| Transaction | Group of database operations managed together |
| Commit | Make the transaction's changes permanent |
| Rollback | Cancel uncommitted changes |
| Parameterized query | SQL statement with values supplied separately |

## 4. Why begin with SQLite?

SQLite is included with standard Python installations through the `sqlite3` module. It stores a database in a local file, so students can practise without installing a database server or creating credentials.

SQLite is excellent for this lesson's local exercises. Production teams may use PostgreSQL, MySQL or managed database services; their drivers and operational setup differ.

## 5. Connect to a local SQLite database

```python
import sqlite3

connection = sqlite3.connect("retail_orders.db")

print("Database connection created")
```

**Expected output**

```text
Database connection created
```

**Explanation**

- `import sqlite3` loads Python's built-in SQLite interface.
- `connect()` opens the database file or creates it if it does not exist.
- `connection` represents the database session.

The database file is created in the current working directory. Use `Path.cwd()` if you need to confirm where it was created.

## 6. Create a cursor

```python
cursor = connection.cursor()

print(type(cursor).__name__)
```

**Expected output**

```text
Cursor
```

The cursor executes SQL and provides methods to retrieve query results. A connection can create cursors as needed.

## 7. Create a table

```python
cursor.execute("""
CREATE TABLE IF NOT EXISTS orders (
    order_id INTEGER PRIMARY KEY,
    customer_name TEXT NOT NULL,
    amount REAL NOT NULL,
    status TEXT NOT NULL
)
""")

connection.commit()

print("Orders table is ready")
```

**Expected output**

```text
Orders table is ready
```

**SQL explanation**

- `CREATE TABLE IF NOT EXISTS` creates the table only if it is absent.
- `order_id` is the primary key.
- `INTEGER`, `TEXT` and `REAL` are SQLite type declarations.
- `NOT NULL` prevents missing values in these columns.
- `commit()` saves the schema change.

## 8. Insert one record

```python
cursor.execute(
    "INSERT OR REPLACE INTO orders (order_id, customer_name, amount, status) VALUES (?, ?, ?, ?)",
    (1001, "Ravi", 1200.50, "PAID")
)

connection.commit()

print("One order inserted")
```

**Expected output**

```text
One order inserted
```

**Important:** The question marks are parameter placeholders in SQLite. Values are supplied as a tuple separately from the SQL statement.

The example uses `INSERT OR REPLACE` to make a notebook cell repeatable for the same primary key. In a production application, choose conflict behavior deliberately; replacing a row can overwrite existing data.

## 9. Insert multiple records

```python
order_rows = [
    (1002, "Anu", 850.00, "PENDING"),
    (1003, "Kiran", 450.00, "PAID"),
    (1004, "Meena", 1500.00, "PAID")
]

cursor.executemany(
    "INSERT OR REPLACE INTO orders (order_id, customer_name, amount, status) VALUES (?, ?, ?, ?)",
    order_rows
)

connection.commit()

print("Rows supplied:", len(order_rows))
```

**Expected output**

```text
Rows supplied: 3
```

`executemany()` runs the same parameterized statement for multiple parameter sets. It is useful for basic bulk inserts.

## 10. Read all records using SELECT

```python
cursor.execute("SELECT order_id, customer_name, amount, status FROM orders ORDER BY order_id")
rows = cursor.fetchall()

for row in rows:
    print(row)
```

**Expected output**

```text
(1001, 'Ravi', 1200.5, 'PAID')
(1002, 'Anu', 850.0, 'PENDING')
(1003, 'Kiran', 450.0, 'PAID')
(1004, 'Meena', 1500.0, 'PAID')
```

**Explanation**

- `execute()` sends the SELECT statement to SQLite.
- `fetchall()` retrieves all remaining result rows into a Python list.
- Each row is a tuple in the selected column order.
- `ORDER BY` makes the output order explicit.

**Memory note:** `fetchall()` is appropriate for this small practice table. For large tables, use bounded batches or database-side filtering.

## 11. Fetch one row or a limited number

### 11.1 Fetch one row

```python
cursor.execute("SELECT order_id, customer_name FROM orders ORDER BY order_id")
first_order = cursor.fetchone()

print(first_order)
```

**Expected output**

```text
(1001, 'Ravi')
```

### 11.2 Fetch a bounded number

```python
cursor.execute("SELECT order_id, amount FROM orders ORDER BY order_id")
first_two = cursor.fetchmany(2)

print(first_two)
```

**Expected output**

```text
[(1001, 1200.5), (1002, 850.0)]
```

- `fetchone()` returns the next row or `None`.
- `fetchmany(2)` returns up to two rows.
- `fetchall()` returns every remaining row.

## 12. Filter rows with WHERE

```python
cursor.execute(
    "SELECT order_id, customer_name, amount FROM orders WHERE status = ?",
    ("PAID",)
)

paid_orders = cursor.fetchall()

for order in paid_orders:
    print(order)
```

**Expected output**

```text
(1001, 'Ravi', 1200.5)
(1003, 'Kiran', 450.0)
(1004, 'Meena', 1500.0)
```

The comma in `("PAID",)` makes it a one-item tuple. The query returns only paid orders.

## 13. Why parameterized SQL matters

Never build SQL by inserting untrusted values into the query string.

**Unsafe pattern — do not use:**

```python
customer_name = "Ravi"
unsafe_sql = "SELECT * FROM orders WHERE customer_name = '" + customer_name + "'"
```

This approach becomes dangerous when values contain quotes or malicious SQL text.

**Safe pattern:**

```python
customer_name = "Ravi"

cursor.execute(
    "SELECT order_id, amount FROM orders WHERE customer_name = ?",
    (customer_name,)
)

print(cursor.fetchall())
```

**Expected output**

```text
[(1001, 1200.5)]
```

The database driver binds the value separately. Placeholder syntax differs by driver: SQLite uses `?`; other drivers may use `%s` or named placeholders. Follow the selected driver's documentation.

## 14. Insert, update and delete (CRUD)

CRUD means Create, Read, Update and Delete.

### 14.1 Update a record

```python
cursor.execute(
    "UPDATE orders SET status = ? WHERE order_id = ?",
    ("SHIPPED", 1002)
)

connection.commit()

print("Rows affected:", cursor.rowcount)
```

**Expected output**

```text
Rows affected: 1
```

### 14.2 Verify the update

```python
cursor.execute(
    "SELECT order_id, status FROM orders WHERE order_id = ?",
    (1002,)
)

print(cursor.fetchone())
```

**Expected output**

```text
(1002, 'SHIPPED')
```

### 14.3 Delete a test record

```python
cursor.execute("DELETE FROM orders WHERE order_id = ?", (1004,))
connection.commit()

print("Rows deleted:", cursor.rowcount)
```

**Expected output**

```text
Rows deleted: 1
```

**Production caution:** Always understand the WHERE clause before running UPDATE or DELETE. An omitted filter can change or remove every row.

## 15. Transactions: commit and rollback

A transaction groups operations so that they succeed or fail together.

### 15.1 Demonstrate rollback

```python
try:
    cursor.execute(
        "INSERT INTO orders (order_id, customer_name, amount, status) VALUES (?, ?, ?, ?)",
        (2001, "Test Customer", 300, "PAID")
    )

    cursor.execute(
        "INSERT INTO orders (order_id, customer_name, amount, status) VALUES (?, ?, ?, ?)",
        (1001, "Duplicate ID", 100, "PAID")
    )

    connection.commit()
    print("Transaction committed")

except sqlite3.IntegrityError:
    connection.rollback()
    print("Transaction rolled back")
```

**Expected output**

```text
Transaction rolled back
```

The second insert violates the primary-key constraint. The exception handler rolls back the transaction, so the first insert is also undone.

### 15.2 Verify that the first insert was rolled back

```python
cursor.execute("SELECT COUNT(*) FROM orders WHERE order_id = ?", (2001,))
print(cursor.fetchone()[0])
```

**Expected output**

```text
0
```

**Interpretation:** The first insert did not remain in the database because both operations belonged to the same transaction.

## 16. Use a connection context manager

Python's SQLite connection supports a context manager for transaction handling:

```python
with sqlite3.connect("retail_orders.db") as connection:
    cursor = connection.cursor()
    cursor.execute(
        "UPDATE orders SET status = ? WHERE order_id = ?",
        ("PAID", 1002)
    )

print("Transaction block completed")
```

**Expected output**

```text
Transaction block completed
```

The context manager commits on successful completion and rolls back if an exception escapes the block. It does not mean every resource is necessarily closed at the end of the block in the same way as a file context manager; close the connection explicitly when finished.

## 17. Extract query results into dictionaries

By default, SQLite returns tuples. A row factory can provide named column access.

```python
connection.row_factory = sqlite3.Row
cursor = connection.cursor()

cursor.execute(
    "SELECT order_id, customer_name, amount, status FROM orders ORDER BY order_id"
)

row = cursor.fetchone()

print(row["order_id"])
print(row["customer_name"])
```

**Expected output**

```text
1001
Ravi
```

A named row is convenient when mapping database columns to downstream records. Set the row factory before creating the cursor you intend to use.

## 18. Extract large result sets in batches

Do not automatically call `fetchall()` for a huge table.

```python
cursor.execute("SELECT order_id, amount FROM orders ORDER BY order_id")

total_rows = 0

while True:
    batch = cursor.fetchmany(2)

    if not batch:
        break

    total_rows += len(batch)
    print("Batch size:", len(batch))

print("Total extracted:", total_rows)
```

**Expected output for the current three-row table after the delete**

```text
Batch size: 2
Batch size: 1
Total extracted: 3
```

**Interpretation:** The cursor returns a bounded number of rows each time. This helps control memory usage, although the database and driver may have their own buffering behavior.

## 19. Close database resources

```python
cursor.close()
connection.close()

print("Database resources closed")
```

**Expected output**

```text
Database resources closed
```

Close cursors and connections when they are no longer needed. In production, use context managers or structured cleanup so resources are released even when an error occurs.

## 20. PostgreSQL and MySQL: how production differs

The concepts remain similar, but the driver and SQL placeholder style differ.

| Area | SQLite practice | PostgreSQL example | MySQL example |
|---|---|---|---|
| Driver | Built-in `sqlite3` | `psycopg` | `mysql-connector-python` |
| Server | Local file | Database server/service | Database server/service |
| Placeholder | `?` | Commonly `%s` with psycopg | Driver-specific, commonly `%s` |
| Connection settings | File path | Host, port, dbname, user, password | Host, port, database, user, password |

Install examples:

```bash
python -m pip install "psycopg[binary]"
python -m pip install mysql-connector-python
```

A PostgreSQL connection uses approved host/database/user settings and a secret supplied securely. Never commit credentials. Do not run the remote connection example until the server, account, network access and environment variables have been configured.

## 21. Mini-project: Database Record Extractor

### 21.1 Business requirement

A retail analyst asks for all paid orders above a given amount. Build a small local database, query only the required columns, and export the result to CSV.

### 21.2 Query using a parameter

```python
minimum_amount = 500

cursor.execute(
    """
    SELECT order_id, customer_name, amount, status
    FROM orders
    WHERE status = ? AND amount >= ?
    ORDER BY order_id
    """,
    ("PAID", minimum_amount)
)

extracted_rows = cursor.fetchall()

for row in extracted_rows:
    print(tuple(row))
```

**Expected output for the remaining practice records**

```text
(1001, 'Ravi', 1200.5, 'PAID')
```

The exact rows depend on whether earlier cells were executed. Restart the kernel and rerun the lesson in order for the stated sample state.

### 21.3 Export the extracted rows

```python
import csv

with open("paid_orders_extract.csv", "w", newline="", encoding="utf-8") as file:
    writer = csv.writer(file)
    writer.writerow(["order_id", "customer_name", "amount", "status"])
    writer.writerows(extracted_rows)

print("Extracted rows:", len(extracted_rows))
```

**Expected output**

```text
Extracted rows: 1
```

### 21.4 Verify the output

```python
with open("paid_orders_extract.csv", "r", encoding="utf-8") as file:
    print(file.read())
```

**Expected output**

```text
order_id,customer_name,amount,status
1001,Ravi,1200.5,PAID
```

### 21.5 Project explanation

The Python program connects to SQLite, executes a parameterized query, retrieves only matching rows and writes them to a CSV file. The database performs filtering, so Python does not need to load unrelated records.

## 22. Common errors and troubleshooting

| Error | Likely reason | What to check |
|---|---|---|
| `OperationalError: no such table` | Table was not created in this database file | Check path and run CREATE TABLE |
| `IntegrityError` | Duplicate key or constraint violation | Inspect primary key and constraints |
| `ProgrammingError` | SQL or parameter mismatch | Check placeholders and parameter count |
| Empty result | Filter does not match data | Run a simpler SELECT and inspect values |
| Connection refused | Database server unavailable | Check host, port, service and firewall |
| Authentication failure | Invalid credentials or permissions | Verify secret and database user grants |
| Slow extraction | Too many rows or unindexed filter | Filter in SQL, inspect query plan and fetch in batches |

## 23. Production practices

- Use parameterized SQL for all data values.
- Apply least-privilege database accounts.
- Store credentials outside source code.
- Use transactions for related writes.
- Fetch only required columns and rows.
- Use batch extraction for large datasets.
- Set connection and query timeouts where supported.
- Close connections and cursors reliably.
- Record safe operational metadata; never log passwords.
- Consider isolation, consistency and retry behavior before building production loaders.

## 24. Day 13 revision

- A driver connects Python to a database.
- A connection represents the session; a cursor executes SQL.
- Parameterized queries keep SQL structure separate from supplied values.
- Commit saves changes; rollback cancels uncommitted changes.
- `fetchone()`, `fetchmany()` and `fetchall()` retrieve result rows differently.
- Use SQL filtering and bounded batches for larger extracts.
- Day 14 uses Pandas to inspect and transform tabular datasets.


---

# Beginner Teaching Extension — Day 13

## A. Understand the connection parts

Python does not speak directly to every database protocol. A database driver provides the communication layer. A connection represents a session with the database. A cursor executes SQL and retrieves results. A transaction groups changes that should succeed or fail together.

## B. Create a local SQLite database

SQLite is included with Python and stores a database in a local file, making it useful for beginner practice. It is not the same deployment model as a shared PostgreSQL or MySQL server.

```python
import sqlite3

connection = sqlite3.connect("shop.db")
print("Database connection created")
connection.close()
```

Expected output: Database connection created. connect() opens or creates the database file. close() releases the connection.

## C. Create a table and insert rows

```python
import sqlite3

connection = sqlite3.connect("shop.db")
cursor = connection.cursor()
cursor.execute("CREATE TABLE IF NOT EXISTS orders (order_id INTEGER PRIMARY KEY, customer TEXT, amount REAL)")
cursor.execute("INSERT INTO orders (order_id, customer, amount) VALUES (1, 'Asha', 500)")
connection.commit()
print("Table ready and row inserted")
connection.close()
```

Expected output: Table ready and row inserted. execute() runs SQL. commit() makes the transaction's changes durable. Running the INSERT again with the same primary key will fail because the key must be unique; reset the sample database or use a deliberate upsert strategy for repeatable practice.

## D. Query and fetch records

```python
import sqlite3

connection = sqlite3.connect("shop.db")
cursor = connection.cursor()
cursor.execute("SELECT order_id, customer, amount FROM orders ORDER BY order_id")
rows = cursor.fetchall()
for row in rows:
    print(row)
connection.close()
```

Expected first-run output:

```text
(1, 'Asha', 500.0)
```

fetchall() returns all selected rows as a list of tuples. For a large result, fetching all rows may consume significant memory; use fetchmany() or iteration when appropriate.

## E. Parameterized queries

Never build SQL by joining untrusted text into the query string. Use placeholders:

```python
import sqlite3

connection = sqlite3.connect("shop.db")
cursor = connection.cursor()
customer_name = "Ravi"
amount = 800
cursor.execute("INSERT INTO orders (order_id, customer, amount) VALUES (?, ?, ?)", (2, customer_name, amount))
connection.commit()
print("Parameterized insert completed")
connection.close()
```

Expected output: Parameterized insert completed. The question-mark placeholders are SQLite syntax. Other database drivers may use different placeholder styles. Parameters are supplied separately from SQL text, which helps prevent SQL injection and handles quoting correctly.

## F. Update and delete carefully

```python
cursor.execute("UPDATE orders SET amount = ? WHERE order_id = ?", (550, 1))
connection.commit()
```

Before a DELETE or UPDATE in a real system, inspect the WHERE condition and affected row count. An omitted or incorrect WHERE clause can change every row.

## G. Transaction rollback

```python
connection = sqlite3.connect("shop.db")
cursor = connection.cursor()
try:
    cursor.execute("UPDATE orders SET amount = ? WHERE order_id = ?", (999, 1))
    raise ValueError("Simulated failure before commit")
    connection.commit()
except ValueError:
    connection.rollback()
    print("Changes rolled back")
finally:
    connection.close()
```

Expected output: Changes rolled back. rollback() cancels uncommitted transaction changes. This is a teaching simulation; do not use an artificial raise in normal production business logic.

## H. Mini-project: Database Record Extractor

Create a local orders table, insert a few sample rows using parameterized statements, select records, and print them. Compare the number of inserted rows with the number retrieved. Test an invalid duplicate key and explain why the database rejects it. Ensure the connection is closed after work.

## I. Common mistakes

- Forgetting commit after changes.
- Forgetting to close the connection.
- Building SQL by string concatenation with user input.
- Calling fetchall() for a result too large for memory.
- Re-running sample INSERT statements with duplicate primary keys.
- Assuming SQLite file behavior is identical to a managed PostgreSQL/MySQL service.

## J. Readiness check

The student can explain driver, connection, cursor, query, result row, commit, rollback and parameterized query. They can create a local SQLite table, insert and retrieve rows, and explain why SQL parameters matter.
