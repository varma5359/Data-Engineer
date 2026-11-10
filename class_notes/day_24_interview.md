# Day 24 Interview Preparation: Transactions and ACID Principles

**Phase:** 3 — SQL and Database Engineering  
**Level:** Beginner to junior Data Engineer  
**Mini-project:** Transaction Simulation  
**Database:** PostgreSQL

---

## 1. How to answer transaction questions

Use a business operation such as a bank transfer or order placement. Explain what can go wrong if only part of the operation succeeds, then connect the answer to the transaction command or ACID property.

---

## 2. Core questions with complete answers

### 1. What is a transaction?

A transaction is a group of database operations treated as one logical unit. It should complete successfully as a whole or have its uncommitted changes cancelled. A bank transfer is a classic example because both the debit and credit must be handled together.

### 2. What does BEGIN do?

BEGIN starts an explicit transaction in PostgreSQL. Statements after BEGIN belong to that transaction until COMMIT or ROLLBACK.

### 3. What does COMMIT do?

COMMIT completes the current transaction and makes its changes permanent according to the database's durability guarantees. Other sessions can then see the committed changes according to their isolation and snapshot rules.

### 4. What does ROLLBACK do?

ROLLBACK cancels the uncommitted changes in the current transaction and returns the database to the state before that transaction began.

### 5. Can ROLLBACK undo a committed transaction?

No. Once committed, the transaction is complete. To correct a committed business mistake, use a compensating transaction or an appropriate recovery procedure. A normal ROLLBACK cannot reverse a completed commit.

### 6. What is SAVEPOINT?

SAVEPOINT creates a named point inside a transaction. ROLLBACK TO SAVEPOINT cancels changes made after that point while keeping earlier transaction work.

### 7. What is ACID?

ACID stands for Atomicity, Consistency, Isolation and Durability. These properties describe important guarantees of reliable transaction processing.

### 8. Explain atomicity.

Atomicity means all required operations in a transaction succeed together or are rolled back together. In a transfer, the debit and credit should not be left half-complete.

### 9. Explain consistency.

Consistency means a transaction respects database constraints and required business rules, moving the database from one valid state to another. A CHECK constraint that prevents negative account balances is one example.

### 10. Explain isolation.

Isolation controls how concurrent transactions interact. It prevents transactions from seeing or producing certain inconsistent intermediate results. The exact guarantees depend on the selected isolation level and database engine.

### 11. Explain durability.

Durability means committed changes are preserved according to the database's durability configuration, even after a crash. Write-ahead logging and recovery mechanisms support this. Backups remain necessary for disaster recovery.

### 12. What is autocommit?

Autocommit means a successful statement outside an explicit transaction is committed automatically. Applications can also explicitly begin a transaction to group multiple statements.

### 13. What is a dirty read?

A dirty read occurs when one transaction reads another transaction's uncommitted change. If the writer rolls back, the reader has used data that never became committed. PostgreSQL does not allow dirty reads; READ UNCOMMITTED behaves like READ COMMITTED.

### 14. What is a non-repeatable read?

A transaction reads the same row twice and gets different committed values because another transaction updated and committed that row between the reads.

### 15. What is a phantom read?

A transaction repeats a query for a set of rows and sees a different set because another transaction inserted or deleted rows that match the query predicate.

### 16. What is a lost update?

A lost update happens when concurrent operations overwrite one another's changes because they use stale values or an unsafe read-modify-write pattern. Use atomic updates, locking or suitable isolation and retry logic.

### 17. What are transaction isolation levels?

Common SQL isolation levels are READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ and SERIALIZABLE. They offer progressively stronger rules for what concurrent transactions can observe, although exact behavior varies across database systems.

### 18. Which isolation levels does PostgreSQL support?

PostgreSQL supports READ COMMITTED, REPEATABLE READ and SERIALIZABLE. It treats READ UNCOMMITTED as READ COMMITTED. PostgreSQL uses MVCC for concurrency control.

### 19. What is MVCC?

Multi-Version Concurrency Control stores or manages multiple row versions so transactions can read a consistent snapshot while writes occur. It reduces reader-writer blocking for many ordinary operations. It does not eliminate all locking or conflicts.

### 20. What is SELECT FOR UPDATE?

In PostgreSQL, SELECT ... FOR UPDATE locks selected rows against conflicting updates and locking reads until the transaction ends. It is useful when an application must read a row and then safely update it.

### 21. What is a deadlock?

A deadlock occurs when transactions wait for locks held by one another in a cycle. The database detects the cycle and aborts one transaction so the other can proceed.

### 22. How can you reduce deadlocks?

Keep transactions short, access rows in a consistent order, avoid unnecessary locks and retry transactions safely when a deadlock occurs.

### 23. Why should transactions be short?

Long transactions hold locks and snapshots for longer, increasing contention and resource use. They can delay other operations and increase the impact of failures.

### 24. What happens if a statement fails in a PostgreSQL transaction?

The transaction enters an aborted state. Normal statements cannot continue until the transaction is rolled back or rolled back to a suitable savepoint. A Python application must explicitly handle this state.

### 25. What is the difference between ROLLBACK and ROLLBACK TO SAVEPOINT?

ROLLBACK cancels the entire current transaction. ROLLBACK TO SAVEPOINT cancels only work after the named savepoint and keeps earlier work in the transaction.

### 26. Does a transaction automatically enforce every business rule?

No. Database constraints enforce rules declared in the database, but some business rules require application logic, conditional updates, locks or additional validation. A transaction provides the boundary for coordinating the work.

### 27. How would you implement a bank transfer?

Start a transaction, debit the source account only if it has sufficient funds, verify that exactly one row was updated, credit the destination account, verify the result and commit. If any step fails, roll back. A production design must also consider concurrent transfers, idempotency and audit records.

### 28. Why use a conditional UPDATE for a debit?

A condition such as balance >= transfer_amount prevents the update from reducing the account below the allowed balance. The application must still check the affected row count to know whether the debit actually happened.

### 29. What is a transaction boundary?

A transaction boundary defines which operations must succeed or fail together. A loader might treat one validated batch as a transaction, while a multi-step order operation might treat order and inventory changes as one transaction.

### 30. Should a very large data load be one transaction?

Not always. One huge transaction can hold locks, use resources and increase recovery cost. Batch transactions and staging tables may be more appropriate. The correct batch size depends on atomicity requirements, volume, database capacity and recovery design.

### 31. What is the difference between a database transaction and a backup?

A transaction controls a unit of database work. A backup is a copy or recovery artifact used to restore data after loss or corruption. Transactions do not replace backups.

### 32. What is a compensating transaction?

It is a new business operation that reverses or offsets a previously committed operation. For example, a refund may compensate for a completed payment. It is not the same as rolling back the original committed transaction.

---

## 3. Scenario-based interview questions

### Scenario 1: A transfer debits Asha but does not credit Ravi

**Good answer:** The transfer was not treated as one atomic unit or the failure was not handled correctly. I would group both updates in one transaction, verify the debit affected exactly one row, perform the credit, and commit only if all required operations succeed. Otherwise I would roll back.

### Scenario 2: A Python loader gets a constraint error and then every later statement fails

**Good answer:** In PostgreSQL the transaction is in an aborted state after the error. The application must roll back before issuing new statements. I would also record the failed batch and decide whether to reject it or retry after correcting the data.

### Scenario 3: Two users withdraw money at the same time

**Good answer:** I would use a conditional update, row locking or suitable isolation to prevent both transactions from spending the same available balance. I would check affected rows, keep the transaction short and implement safe retry handling for retryable conflicts.

### Scenario 4: A transaction waits for a long time

**Good answer:** I would inspect blocking sessions and locks, identify the transaction holding the conflicting lock, and check whether it is idle or performing long work. I would avoid leaving transactions open and make sure application code commits or rolls back promptly.

### Scenario 5: The database reports a deadlock

**Good answer:** One transaction was aborted by deadlock detection. I would inspect the involved statements and lock order, standardize row access order and retry the entire transaction safely if the error is retryable.

### Scenario 6: The business asks for SERIALIZABLE isolation everywhere

**Good answer:** SERIALIZABLE offers strong guarantees but can cause transaction aborts that require retries. I would choose isolation based on business correctness, contention and performance, test the workload, and ensure the application handles serialization failures.

### Scenario 7: A batch loader fails after inserting 70 percent of the records

**Good answer:** I would define whether the batch must be all-or-nothing. If so, the batch should be in one transaction or staged and published atomically. If smaller batch commits are used, the pipeline needs checkpoints, idempotent keys and a recovery plan so reruns do not duplicate data.

---

## 4. Mini-project defence

Be ready to explain:

1. Why is a transfer a transaction?
2. What would happen if the second update failed?
3. How did you prove ROLLBACK restored the previous balance?
4. What is the difference between COMMIT and SAVEPOINT?
5. How does the CHECK constraint protect the balance?
6. What happens when two sessions update the same row?
7. What does SELECT FOR UPDATE protect?
8. What is a deadlock?
9. Why should the transaction be short?
10. How would you make a data loader safe after a partial failure?

---

## 5. Common interview traps

- Claiming rollback can undo committed data.
- Saying ACID consistency means every business rule is automatically known to the database.
- Claiming SERIALIZABLE can never fail.
- Assuming PostgreSQL permits dirty reads.
- Forgetting to roll back after a PostgreSQL transaction error.
- Ignoring affected-row counts on conditional updates.
- Holding locks while calling external services.
- Retrying a transaction without protecting external side effects.
- Treating backups and transactions as the same thing.

---

## 6. Readiness checklist

- [ ] I can explain BEGIN, COMMIT and ROLLBACK.
- [ ] I can use SAVEPOINT.
- [ ] I can explain all ACID properties with examples.
- [ ] I can describe dirty, non-repeatable and phantom reads.
- [ ] I can explain PostgreSQL's isolation behavior at a high level.
- [ ] I understand row locks and SELECT FOR UPDATE.
- [ ] I can explain deadlocks and safe retries.
- [ ] I can design a transaction boundary for a data load.
