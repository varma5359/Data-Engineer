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
