# Day 29 Interview Preparation: Python and Database Integration

**Phase:** 3 — SQL and Database Engineering  
**Level:** Beginner to junior Data Engineer  
**Mini-project:** Database Loader  
**Python driver:** Psycopg 3  
**Database:** PostgreSQL

---

## 1. Core questions with complete answers

### 1. Why does a Data Engineer connect Python to a database?

Python can read files, call APIs, validate and transform records. A relational database stores structured data, enforces constraints and supports SQL queries. Python coordinates extraction and loading while the database manages durable structured storage.

### 2. What is a database driver?

A database driver is a library that allows an application to communicate with a database using its protocol. Psycopg is a PostgreSQL driver for Python.

### 3. What is Psycopg?

Psycopg is a Python adapter for PostgreSQL. It allows Python applications to connect, execute SQL statements, pass parameters and retrieve results.

### 4. What is the difference between Psycopg 2 and Psycopg 3?

They are different major versions of the PostgreSQL adapter with API and behavior differences. The course uses Psycopg 3, so code and installation instructions should consistently follow version 3 documentation.

### 5. What is a database connection?

A connection is an active session between the Python application and the database server. It holds connection state and is used to create cursors and manage transactions.

### 6. What is a cursor?

A cursor is an object used to execute SQL statements and retrieve query results. A cursor belongs to a connection.

### 7. What does cursor.execute do?

It sends a SQL statement to PostgreSQL, with any supplied parameter values. It can execute SELECT, INSERT, UPDATE, DELETE and DDL statements, subject to permissions.

### 8. What is a parameterized query?

A parameterized query keeps SQL structure separate from data values. Psycopg uses %s placeholders for values and receives the values separately.

```python
cursor.execute(
    "SELECT * FROM orders WHERE order_id = %s",
    (1001,)
)
```

This is safer and clearer than constructing SQL with string concatenation.

### 9. Why should SQL values not be concatenated into query strings?

Untrusted text can change the meaning of the SQL statement and create SQL injection vulnerabilities. Parameterized queries send values separately from SQL syntax.

### 10. Why does a one-value tuple need a comma?

In Python, parentheses alone do not make a tuple. (1001,) is a one-element tuple. (1001) is just an integer expression.

### 11. What is the difference between connection and cursor?

The connection represents the database session and transaction context. The cursor executes statements and retrieves results within that connection.

### 12. What does fetchone do?

fetchone returns the next row from the result set, or None when there are no more rows.

### 13. What does fetchmany do?

fetchmany retrieves up to a specified number of rows from the result set. It can be used to process results in smaller groups.

### 14. What does fetchall do?

fetchall retrieves all remaining result rows into Python memory. It is convenient for small result sets but can use too much memory for very large queries.

### 15. What is executemany?

executemany executes the same parameterized statement for multiple parameter sets. It is useful for inserting or updating a batch of records. For high-volume loading, COPY may be more efficient.

### 16. What is a transaction in Python database code?

A transaction groups database statements so they commit together or roll back together. Psycopg connections manage transaction behavior and provide context-manager support.

### 17. How does a Psycopg connection context manager work?

When used as a context manager, a connection commits the transaction on successful exit and rolls it back if an exception escapes the block. The connection is then closed when the context exits.

### 18. What happens if an SQL statement fails inside a PostgreSQL transaction?

The transaction enters an aborted state. Further normal statements fail until the transaction is rolled back or rolled back to a savepoint. Python code must handle this explicitly or use a context manager that rolls back on exception.

### 19. What is commit?

Commit completes the transaction and makes its changes permanent according to database guarantees.

### 20. What is rollback?

Rollback cancels uncommitted changes in the current transaction.

### 21. What is an upsert?

An upsert inserts a row if the key does not exist, or updates the existing row when a specified conflict occurs. PostgreSQL supports INSERT ... ON CONFLICT.

### 22. What is idempotency in a data pipeline?

Idempotency means repeating the same logical operation does not create unintended repeated effects. A rerunnable loader may use stable keys and upsert logic so the same source record does not create duplicate target rows.

### 23. Is ON CONFLICT DO UPDATE always the correct choice?

No. It depends on business rules. Some duplicates should be rejected, ignored, updated or historized. An upsert can overwrite newer values if the source is stale, so update conditions and source timestamps may be needed.

### 24. Why validate data before loading?

Validation catches missing keys, invalid amounts, unsupported statuses, duplicate identifiers and malformed values before they affect downstream reports or applications. Database constraints provide a second layer of protection.

### 25. What is batch loading?

Batch loading processes records in controlled groups rather than one at a time or all at once. It balances memory, network, throughput and transaction recovery requirements.

### 26. What is the difference between executemany and COPY?

executemany runs a statement for multiple parameter sets. COPY is a database bulk-transfer mechanism designed for high-throughput loading and is often preferable for very large datasets. The best approach depends on the workload and validation process.

### 27. Why not fetch a million rows with fetchall?

fetchall stores all remaining rows in Python memory. A very large result can exhaust memory. Use controlled fetching, server-side cursor approaches or database-side transformations as appropriate.

### 28. Why use Decimal for financial values?

PostgreSQL NUMERIC maps naturally to Python Decimal. Decimal represents base-10 decimal values more predictably than binary floating-point for financial calculations.

### 29. Why keep database passwords out of Python source code?

Source code may be committed, copied or shared. Credentials should come from secure environment configuration or a secret manager, and should not appear in logs or screenshots.

### 30. What is a connection pool?

A connection pool maintains a controlled set of reusable database connections. It reduces connection setup overhead and helps prevent applications from opening too many sessions.

### 31. What is connection leakage?

A connection leak occurs when application code fails to close or return connections to the pool. Over time, the database may run out of available connections.

### 32. How do you handle a connection failure?

Check host, port, database name, username, password, service status, network/firewall, TLS and database permissions. Capture the exact exception without exposing secrets. Retry only transient failures with controlled backoff.

### 33. Why should a loader verify after insertion?

A successful INSERT command does not prove the full business load is correct. Verify row counts, key uniqueness, required values and aggregate totals against the source batch.

### 34. What is a rejected-record flow?

Invalid records are separated from accepted records and stored with a reason for investigation. This prevents bad records from silently entering trusted tables and helps operations correct source problems.

### 35. What is the difference between a database constraint and Python validation?

Python validation catches issues before sending records. Database constraints enforce integrity for every writer, including applications other than this Python script. Use both layers where appropriate.

### 36. How do you make a loader recoverable?

Define batch boundaries, use transactions, stable business keys, idempotent writes, rejected-record handling, checkpoints and reconciliation. A retry should not duplicate or corrupt previously loaded data.

---

## 2. Scenario-based interview questions

### Scenario 1: Your loader inserts 500 rows and fails on row 501

**Good answer:** I would first determine the transaction boundary. If all 501 operations are in one transaction, the failed batch can be rolled back. If earlier batches were already committed, I would use stable keys and checkpoints to resume safely. I would capture the rejected record and its error reason.

### Scenario 2: The script runs twice and creates duplicate rows

**Good answer:** The loader is not idempotent. I would identify the stable business key, enforce uniqueness in the target database and implement the correct conflict behavior, such as reject, ignore or upsert. I would test reruns and ensure stale source data cannot overwrite newer target data.

### Scenario 3: A SQL injection vulnerability is found

**Good answer:** I would remove string concatenation for values and use parameterized SQL. Dynamic table or column identifiers must be composed with safe identifier tools, not treated as value parameters. I would review logs and access to determine whether the vulnerability was exploited.

### Scenario 4: The Python process uses too much memory

**Good answer:** I would check whether it uses fetchall or loads the entire source file into a list. I would process records in batches, use streaming or server-side cursors where suitable, and push transformations into SQL when appropriate.

### Scenario 5: PostgreSQL reports the transaction is aborted

**Good answer:** A previous statement failed. I would roll back the transaction or roll back to a savepoint before continuing. I would also identify the original constraint or SQL error rather than suppressing it.

### Scenario 6: The loader says four rows inserted but the report total is wrong

**Good answer:** I would verify the exact target rows, check numeric types and rounding, validate the source-to-target mapping, inspect duplicate/upsert behavior and compare count and amount totals. I would not rely on rowcount alone as a complete correctness check.

### Scenario 7: The source sends the same order ID with a different amount

**Good answer:** I would clarify whether the new record is a correction, a stale replay or an invalid duplicate. Then choose reject, update, version history or an upsert with source timestamp rules. A primary key conflict alone does not tell us the correct business action.

---

## 3. Mini-project defence

Be ready to explain:

1. Why did you choose Psycopg 3?
2. How did you configure the database connection?
3. What is the difference between a connection and cursor?
4. Why did you use parameterized SQL?
5. How did you insert one row and multiple rows?
6. How did the connection context manager handle a failure?
7. How did you verify that rollback removed the test record?
8. How did you validate incoming records?
9. How did you make the loader safe to rerun?
10. How did you verify the loaded row count and total amount?
11. How would you handle millions of records?
12. How would you store credentials in production?

---

## 4. Common interview traps

- Confusing Psycopg with the database server.
- Confusing connection and cursor.
- Concatenating user values into SQL.
- Forgetting the comma in a one-element tuple.
- Assuming fetchall is safe for any data size.
- Assuming executemany is the fastest bulk-loading method.
- Forgetting rollback after a PostgreSQL error.
- Treating rowcount as the only validation.
- Using upsert without defining the business conflict rule.
- Hardcoding credentials in source control.
- Retrying a non-idempotent load blindly.
- Ignoring numeric precision for financial amounts.

---

## 5. Readiness checklist

- [ ] I can explain database drivers and Psycopg 3.
- [ ] I can configure and test a PostgreSQL connection.
- [ ] I understand connection and cursor.
- [ ] I can execute parameterized SQL.
- [ ] I can fetch one, many or all rows appropriately.
- [ ] I can insert a batch.
- [ ] I understand commit and rollback.
- [ ] I can explain connection context-manager behavior.
- [ ] I can validate records before loading.
- [ ] I can use an appropriate idempotency strategy.
- [ ] I can verify counts and totals.
- [ ] I can explain how I would scale to large datasets.
