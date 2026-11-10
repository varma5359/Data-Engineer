# Day 13 Interview Preparation: Python Database Connectivity

**Mini-project:** Database Record Extractor

## 1. Core questions

1. **Why connect Python to a database?**  
   To extract, load, validate and automate database operations as part of data pipelines.

2. **What is a database driver?**  
   A library that allows Python to communicate with a database engine.

3. **What is a connection?**  
   An active session between the Python application and the database.

4. **What is a cursor?**  
   An object used to execute SQL statements and retrieve results.

5. **What is a transaction?**  
   A group of operations managed as a unit, with commit or rollback behavior.

6. **What does commit do?**  
   It makes the current transaction's changes permanent.

7. **What does rollback do?**  
   It cancels uncommitted changes in the current transaction.

8. **Why parameterize SQL?**  
   To bind values separately from SQL text, reducing SQL injection risk and handling values safely.

9. **What is the difference between fetchone and fetchall?**  
   fetchone returns the next row; fetchall retrieves all remaining rows.

10. **Why avoid fetchall on a huge table?**  
    It can load too many rows into application memory. Use filtering and bounded batches.

11. **Why use SQLite in a beginner lesson?**  
    It is included with Python and runs locally without a separate database server.

12. **Is SQLite identical to PostgreSQL or MySQL?**  
    No. SQL dialects, concurrency, data types, drivers and server operations differ.

## 2. Difference-based questions

| Compare | Answer |
|---|---|
| Connection vs cursor | Session to database vs object executing statements |
| Commit vs rollback | Save transaction vs cancel uncommitted changes |
| fetchone vs fetchmany | One row vs bounded number of rows |
| fetchmany vs fetchall | Batch of rows vs all remaining rows |
| SQL string concatenation vs parameterized SQL | Unsafe value construction vs separate bound parameters |
| SQLite vs PostgreSQL | Local embedded database vs server-based relational database |

## 3. Practical questions

13. **How do you connect to SQLite?**  
    Import sqlite3 and call `sqlite3.connect("database_file.db")`.

14. **How do you create a table?**  
    Execute a CREATE TABLE statement through a cursor and commit the transaction.

15. **How do you insert multiple records?**  
    Use `executemany()` with a parameterized INSERT statement and a sequence of parameter tuples.

16. **How do you retrieve paid orders?**  
    Execute a SELECT with a WHERE condition and bind the status value as a parameter.

17. **How do you avoid SQL injection?**  
    Never concatenate untrusted values into SQL. Use the database driver's parameter placeholders.

18. **How do you extract a large table?**  
    Filter in SQL, select required columns and fetch in batches or use keyset pagination.

19. **How do you protect database credentials?**  
    Use environment variables or a secret manager, least-privilege accounts and secure network connections.

## 4. Scenario questions

### Scenario 1: Duplicate primary-key error

**Problem:** An INSERT fails because the order ID already exists.

**Strong answer:** Inspect the source key and define duplicate behavior. Depending on the requirement, reject the row, update an existing record or use a database-specific upsert. Do not blindly replace records.

### Scenario 2: Pipeline inserts 1,000 rows, then fails

**Problem:** Some records may have been inserted before the failure.

**Strong answer:** Use a transaction for the logical unit of work. Roll back if the batch fails, or use smaller committed batches with a checkpoint and idempotent load design.

### Scenario 3: Query is slow and returns millions of rows

**Problem:** Python calls fetchall on a full table.

**Strong answer:** Select only required columns, filter at the database, inspect query performance and fetch in bounded batches. Consider indexing the filter/join columns based on workload.

### Scenario 4: A query is vulnerable to SQL injection

**Problem:** The code builds SQL by concatenating a user-supplied customer name.

**Strong answer:** Replace string concatenation with a parameterized query. The driver binds the value separately from SQL syntax.

### Scenario 5: Connection refused

**Problem:** Python cannot connect to a PostgreSQL server.

**Strong answer:** Check that the server is running, hostname and port are correct, network/firewall rules allow access, credentials are valid and the database user has permission.

## 5. Troubleshooting questions

20. **What causes a no-such-table error?**  
    The table may not exist in the database file being opened, or the script may be using a different working directory/database path.

21. **Why can a SELECT return no rows?**  
    The filter may not match, values may have unexpected casing/whitespace, or the table may be empty.

22. **Why close a connection?**  
    To release database resources and avoid exhausting connection limits.

23. **Why should SQL values not be printed into logs?**  
    They may contain personal or confidential data. Log safe identifiers and counts instead.

## 6. Mini-project explanation

"I created a local SQLite database, created an orders table, inserted records, queried paid orders using parameters and exported the selected result to CSV. I used commit and rollback to understand transaction behavior and fetched rows in batches to understand memory management."

## 7. Follow-up questions

- What is connection pooling and why is it useful?
- What is an upsert?
- What happens if commit is not called?
- How do you resume a failed bulk extraction?
- How do you prevent duplicate loads?
- How do indexes affect extraction queries?

## 8. Quick revision

- Driver communicates with the database.
- Connection manages the session; cursor executes SQL.
- Parameterize all values.
- Use transactions for related writes.
- Filter and batch large extracts.
- Protect credentials and close resources.


---

## 7. Follow-up interview questions

16. What is a database driver? A library that implements communication between an application and a database.
17. What is a connection? An active session through which an application communicates with the database.
18. What is a cursor? An object used by many database APIs to execute statements and retrieve results.
19. What does commit do? Makes the current transaction's changes durable.
20. What does rollback do? Cancels uncommitted transaction changes.
21. Why use parameterized SQL? It separates values from SQL syntax, handles data safely and helps prevent SQL injection.
22. What is SQL injection? A vulnerability where untrusted input changes the meaning of a SQL statement.
23. fetchone versus fetchall? fetchone returns one row; fetchall returns all remaining rows.
24. Why can fetchall be risky for huge results? It materializes all returned rows in memory.
25. What is SQLite? An embedded relational database commonly stored in a local file.
26. Is SQLite identical to PostgreSQL? No. They differ in deployment, concurrency, features, types and SQL/driver behavior.
27. Why close a connection? To release resources and avoid leaving sessions open.
28. What happens if a primary key is duplicated? The database rejects the row because the key must be unique.
29. Why inspect affected rows after UPDATE? To confirm the intended records changed and detect an incorrect filter.
30. What is a transaction? A logical unit of database work that is committed or rolled back.

## 8. Scenario questions

The script inserts duplicate records on every run. Define a stable key and an intentional idempotent load strategy, such as checking existing records or using a database-supported upsert.

A query returns millions of rows. Avoid fetching everything at once; select only needed columns and retrieve in manageable batches.

The script builds SQL using a customer-supplied name. Replace string concatenation with parameterized SQL.

## 9. Mini-project defense

- Which database did you use and why?
- What does the driver do?
- How did you protect values in SQL statements?
- When did you commit or roll back?
- How would you adapt the code to PostgreSQL or MySQL?

## 10. Readiness rubric

| Skill | Ready when the student can… |
|---|---|
| Connection | Open and close a database connection |
| SQL execution | Create, insert and select records |
| Parameters | Execute a parameterized query |
| Transactions | Explain commit and rollback |
| Results | Retrieve and inspect rows |
| Scale | Explain why large results need batching |
