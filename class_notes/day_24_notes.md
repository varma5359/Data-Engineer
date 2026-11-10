# Day 24: Transactions and ACID Principles

**Pathway:** TechPhile.ai — Data Engineer Career Pathway  
**Phase:** 3 — SQL and Database Engineering  
**Audience:** Freshers, college students and junior Data Engineer candidates  
**Level:** Beginner to junior | **Class time:** 2–3 hours  
**Approved syllabus:** Transactions, COMMIT, ROLLBACK, ACID, isolation levels, concurrency, transaction consistency  
**Mini-project:** Transaction Simulation  
**Primary database:** PostgreSQL; MySQL differences are identified where relevant.

---

## 1. How to study

Read one concept, predict what the query or program will do, execute the small example, compare the result and explain it aloud. Do not run the entire lesson as one large script.

## 2. Prerequisite and learning outcomes

Use the TechStore training database from Day 20: customers, products, orders and order_items. Confirm you are connected to the correct practice database. If sample data differs, inspect it before comparing outputs. Never experiment on production data.

By the end, you should understand each item in today's syllabus, execute the main examples, explain the output, recognize common mistakes and complete **Transaction Simulation**.

## 3. What is a transaction?

A transaction is a logical unit of database work. Example: a transfer debits one account and credits another. If one step fails, the database should not commit a half-finished operation.

## 4. Disposable practice table

```sql
CREATE TABLE demo_accounts(
 account_id INTEGER PRIMARY KEY,
 account_holder VARCHAR(80) NOT NULL,
 balance NUMERIC(12,2) NOT NULL CHECK(balance>=0)
);
INSERT INTO demo_accounts VALUES(1,'Asha',5000),(2,'Ravi',3000);
SELECT * FROM demo_accounts ORDER BY account_id;
```
Expected: Asha 5000, Ravi 3000. Use training data only.

## 5. BEGIN, COMMIT, ROLLBACK

```sql
BEGIN;
UPDATE demo_accounts SET balance=balance-500 WHERE account_id=1;
UPDATE demo_accounts SET balance=balance+500 WHERE account_id=2;
SELECT * FROM demo_accounts ORDER BY account_id;
```
Inside this session balances are 4500 and 3500. Other sessions normally do not see uncommitted changes under PostgreSQL READ COMMITTED.

```sql
COMMIT;
```
COMMIT ends the transaction and makes changes durable according to database configuration. Verify balances.

Test rollback:
```sql
BEGIN;
UPDATE demo_accounts SET balance=balance-200 WHERE account_id=1;
UPDATE demo_accounts SET balance=balance+200 WHERE account_id=2;
ROLLBACK;
```
Balances remain 4500 and 3500. ROLLBACK cancels uncommitted work; it cannot undo a committed transaction. Use an approved compensating transaction or recovery process.

## 6. SAVEPOINT

A savepoint marks a position inside a transaction:
```sql
BEGIN;
UPDATE demo_accounts SET balance=balance-100 WHERE account_id=1;
SAVEPOINT after_debit;
UPDATE demo_accounts SET balance=balance+100 WHERE account_id=2;
ROLLBACK TO SAVEPOINT after_debit;
ROLLBACK;
```
Final ROLLBACK cancels all practice work. If you COMMIT instead, the debit remains but credit is undone. A real transfer must never commit this unbalanced state.

## 7. ACID

- Atomicity: all required changes happen together or none.
- Consistency: database rules remain valid.
- Isolation: concurrent work is controlled by isolation guarantees.
- Durability: committed changes persist within configured guarantees.

## 8. Concurrency and isolation levels

Common names: READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE. Behavior differs by DBMS. PostgreSQL treats READ UNCOMMITTED as READ COMMITTED; its REPEATABLE READ uses snapshot isolation; SERIALIZABLE can abort a transaction when a safe serial execution cannot be guaranteed.

Terms:
- Dirty read: read another transaction's uncommitted change.
- Non-repeatable read: reread a row and see a changed committed value.
- Phantom: repeat a range query and see new qualifying rows.
- Lost update: one concurrent write overwrites another's intended change.

## 9. Row locking

Session A:
```sql
BEGIN;
SELECT * FROM demo_accounts WHERE account_id=1 FOR UPDATE;
```
FOR UPDATE locks selected row against conflicting updates/row locks until transaction end.

Session B:
```sql
UPDATE demo_accounts SET balance=balance+100 WHERE account_id=1;
```
Session B waits while A holds the lock. Commit A, then B continues. Keep transactions short; long transactions hold locks and delay maintenance.

## 10. Transactions in pipelines

Begin a transaction, load a bounded batch, validate row counts and constraints, commit only when all checks pass, otherwise rollback and record failure. Check affected row counts. Production also needs idempotency, retry policy and audit evidence.

## 11. Mini-project — Transaction Simulation

Create two practice accounts, commit a 500 transfer, verify balances, rollback a second transfer, demonstrate SAVEPOINT and two-session row locking, and explain ACID. Expected after committed transfer and rolled-back transfer: Asha 4500, Ravi 3500.

**Deliverables:** SQL, before/after balances, screenshots and ACID explanation.

## 12. Self-check

Explain transaction, BEGIN/COMMIT/ROLLBACK, SAVEPOINT, ACID, lost update, isolation, row lock, short transactions and safe batch commit.

## 99. Next lesson

Next: Day 25 — Database Indexing and Query Optimization.