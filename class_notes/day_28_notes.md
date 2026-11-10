# Day 28: Database Administration Fundamentals
**Phase:** 3 — SQL and Database Engineering  
**Mini-project:** Database Operations Checklist  
**Learning time:** 2–3 hours  
**Audience:** Beginners progressing to junior Data Engineer level  
**Databases:** PostgreSQL and MySQL fundamentals; commands are labelled by product.

---

## 1. Why does a Data Engineer need database administration knowledge?

A Data Engineer does more than write SELECT queries. Pipelines depend on databases being available, secure, backed up and correctly configured.

Imagine a daily pipeline fails because:
- The database service is stopped.
- The password or connection details are incorrect.
- The pipeline user lost INSERT permission.
- The database has no available storage.
- Too many connections are open.
- A required table was accidentally deleted.
- A backup exists but nobody has tested restoring it.

A junior Data Engineer may not be the database administrator, but should understand the basic operational checks and know when to escalate.

### Learning outcomes

By the end, you can:
1. Explain basic database administration responsibilities.
2. Distinguish database server, database, schema and user.
3. Understand roles and privileges.
4. Apply least privilege.
5. Explain connection parameters and common connection failures.
6. Inspect active sessions at a basic level.
7. Explain backup versus restore.
8. Understand PostgreSQL logical backups with pg_dump and pg_restore.
9. Understand MySQL backups with mysqldump.
10. Restore a practice database and validate it.
11. Explain why backup verification matters.
12. Prepare a basic database operations checklist.

---

## 2. Understand the database environment

A database application typically connects through a network or local socket to a database server.

Common connection details:
- Host: where the server runs.
- Port: network port used by the database service.
- Database name: the database to connect to.
- Username: account used for authentication.
- Password or another authentication method.
- SSL/TLS options: secure connection settings where required.

Common default ports:
- PostgreSQL: 5432.
- MySQL: 3306.

Defaults can be changed by administrators. Do not assume the default port is always used.

### 2.1 Server versus database

- Database server: the running database software/service.
- Database: a named collection of database objects managed by the server.
- Schema: a namespace organizing objects inside a database in PostgreSQL.
- Table: a database object storing rows and columns.
- Role/user: an identity or permission-bearing principal.

In PostgreSQL, a role can act as a user or group. MySQL has its own user and privilege model.

---

## 3. Identify which database you are connected to

### PostgreSQL

```sql
SELECT current_database();
SELECT current_user;
SELECT version();
```

These show the current database, current role and server version.

### MySQL

```sql
SELECT DATABASE();
SELECT CURRENT_USER();
SELECT VERSION();
```

These show the selected database, authenticated account and server version.

Always verify the environment before running DDL or DML. Accidentally changing production instead of a training database is a serious operational mistake.

---

## 4. Understand roles and permissions

A database role determines what actions an identity can perform.

Common privilege examples:
- CONNECT to a database.
- USAGE on a schema.
- SELECT from a table.
- INSERT into a table.
- UPDATE selected data.
- DELETE rows.
- CREATE objects, where permitted.

A pipeline that only reads source data should not receive unnecessary DELETE or administrative privileges.

### 4.1 Least privilege

Least privilege means granting only the permissions required to perform the job.

Example:
- Reporting user: SELECT access to reporting views.
- Loader user: INSERT/UPDATE access to target tables needed by the pipeline.
- Administrator: controlled administrative privileges.

Do not run every pipeline using a superuser or root account.

---

## 5. PostgreSQL role and privilege lab

Run these commands only if you have permission to create roles. They are administrator operations. If your training environment does not allow them, read the examples and ask your instructor/admin to execute them.

Create a role:

```sql
CREATE ROLE techstore_reader LOGIN PASSWORD 'Use-A-Unique-Training-Password';
```

This creates a login role. The password shown is a placeholder for a disposable lab only. Do not copy it into production or commit credentials to GitHub.

Grant database connection:

```sql
GRANT CONNECT ON DATABASE techstore TO techstore_reader;
```

Connect to the database as an administrator, then grant schema access:

```sql
GRANT USAGE ON SCHEMA public TO techstore_reader;
```

Grant read access to selected tables:

```sql
GRANT SELECT ON TABLE public.customers TO techstore_reader;
GRANT SELECT ON TABLE public.orders TO techstore_reader;
```

The role can read the selected tables, subject to the other database permissions and row-level security rules if configured.

### 5.1 Test permissions

Connect as techstore_reader and try:

```sql
SELECT * FROM public.customers;
```

This should work if the database, schema and table privileges are correctly configured.

Try an UPDATE only in a disposable training environment:

```sql
UPDATE public.customers
SET city = 'Test'
WHERE customer_id = -999;
```

Expected: permission denied if the role has no UPDATE privilege. Even if no row would match, PostgreSQL checks permission to perform the command.

### 5.2 Revoke a permission

As an administrator:

```sql
REVOKE SELECT ON TABLE public.customers FROM techstore_reader;
```

The role no longer has that direct table-level SELECT grant, unless another role or broader privilege grants access.

### 5.3 PostgreSQL default privileges

When new tables are created later, existing grants do not always automatically apply. PostgreSQL supports ALTER DEFAULT PRIVILEGES to define future object grants for objects created by a particular role.

Do not confuse current object privileges with default privileges for future objects.

---

## 6. MySQL user and permission basics

MySQL uses account names that include a host component, for example 'report_user'@'localhost'.

Example administrator commands:

```sql
CREATE USER 'report_user'@'localhost'
IDENTIFIED BY 'Use-A-Unique-Training-Password';

GRANT SELECT
ON techstore.customers
TO 'report_user'@'localhost';
```

Test by connecting as the new user and reading the granted table.

Inspect grants:

```sql
SHOW GRANTS FOR 'report_user'@'localhost';
```

Revoke:

```sql
REVOKE SELECT
ON techstore.customers
FROM 'report_user'@'localhost';
```

The exact host value matters in MySQL account definitions. 'report_user'@'localhost' and 'report_user'@'%' are different account entries.

Do not grant broad access such as ALL PRIVILEGES ON *.* unless the account genuinely needs it and the risk is approved.

---

## 7. Connection management

A pipeline connects to a database using connection parameters and a driver.

Common connection problems:
- Wrong hostname.
- Wrong port.
- Incorrect database name.
- Invalid username or password.
- Database service not running.
- Network/firewall restriction.
- TLS certificate problem.
- Connection limit reached.
- User lacks CONNECT or object privileges.

### 7.1 PostgreSQL connection test with psql

From a terminal:

```bash
psql -h localhost -p 5432 -U postgres -d techstore
```

You will be prompted for the password unless another authentication method is configured.

Useful psql commands:

```text
\conninfo
\l
\dt
\du
\q
```

- \conninfo shows connection details.
- \l lists databases.
- \dt lists tables in the visible schema search path.
- \du lists roles.
- \q exits psql.

These are psql client commands, not SQL statements. They are not run inside every SQL editor.

### 7.2 MySQL connection test

```bash
mysql -h localhost -P 3306 -u root -p
```

After connecting:

```sql
SHOW DATABASES;
USE techstore;
SHOW TABLES;
SELECT DATABASE();
```

The -p option prompts for the password. Avoid placing passwords directly in shell history or shared scripts.

---

## 8. Inspect active connections

### PostgreSQL

```sql
SELECT
    pid,
    usename,
    datname,
    state,
    application_name,
    client_addr
FROM pg_stat_activity
WHERE datname = current_database();
```

This shows sessions visible to the current role. Some session details may be restricted.

Useful state examples:
- active: currently executing a query.
- idle: connected but not currently running a query.
- idle in transaction: transaction is open but no query is currently executing.

An idle-in-transaction session can hold locks and old snapshots. Applications should commit or roll back promptly.

### MySQL

```sql
SHOW PROCESSLIST;
```

This shows server threads/sessions visible to the account. More detailed process information may require privileges.

Do not kill sessions without understanding the query, owner, impact and transaction state.

---

## 9. Backup and restore: understand the difference

A backup is a copy or recovery artifact used to restore data after loss, corruption or an operational mistake.

A restore is the process of rebuilding the database or objects from that backup.

A backup is useful only if:
- It completed successfully.
- It contains the required data and objects.
- It is stored safely.
- It is protected from unauthorized access.
- It can be restored within the required recovery time.
- The restored data passes validation.

**A successful backup command does not prove that the organization can recover. A tested restore is essential.**

### 9.1 Logical versus physical backup

Logical backup:
- Exports SQL or logical object/data representations.
- Can be selective and portable across some environments.
- Common tools: pg_dump, pg_dumpall, mysqldump.

Physical backup:
- Copies database storage files or uses database-native physical backup mechanisms.
- Useful for larger systems and point-in-time recovery strategies.
- Requires compatibility and operational procedures.

This course introduces logical backup and restore. Production physical backup design is a DBA responsibility.

---

## 10. PostgreSQL logical backup with pg_dump

Run this from Windows CMD or a terminal, not inside the SQL query editor.

Custom-format backup:

```bash
pg_dump -h localhost -p 5432 -U postgres -d techstore -F c -f techstore_backup.dump
```

Explanation:
- pg_dump is PostgreSQL's logical backup utility.
- -h specifies host.
- -p specifies port.
- -U specifies user.
- -d specifies database.
- -F c selects custom archive format.
- -f specifies output file.

You may be prompted for a password depending on authentication configuration.

A successful command typically exits without an error and creates the output file. Check the file exists and has a plausible non-zero size. Do not treat file existence alone as proof of recoverability.

### 10.1 Plain SQL backup

```bash
pg_dump -h localhost -p 5432 -U postgres -d techstore -F p -f techstore_backup.sql
```

Plain format creates SQL text that can be restored with psql.

### 10.2 Backup one schema

```bash
pg_dump -h localhost -p 5432 -U postgres -d techstore -n public -F c -f techstore_public.dump
```

-n selects a schema. Schema selection and dependencies must be considered; a partial backup may not contain everything needed by an application.

---

## 11. Restore PostgreSQL backup

For a custom-format archive, pg_restore is used.

Restore into a new empty database:

First create the target database:

```sql
CREATE DATABASE techstore_restore_test;
```

Then run in the terminal:

```bash
pg_restore -h localhost -p 5432 -U postgres -d techstore_restore_test techstore_backup.dump
```

Inspect the restored database:

```bash
psql -h localhost -p 5432 -U postgres -d techstore_restore_test
```

Inside psql:

```text
\dt
```

Then validate:

```sql
SELECT COUNT(*) FROM customers;
SELECT COUNT(*) FROM orders;
```

Compare the counts with the source database. Also check key totals and representative records.

### 11.1 Restore a plain SQL backup

Create a new empty target database, then use:

```bash
psql -h localhost -p 5432 -U postgres -d techstore_restore_test -f techstore_backup.sql
```

Do not restore over a live database without an approved plan. A restore may create, replace or conflict with existing objects depending on the archive and options used.

### 11.2 Important: roles and global objects

pg_dump backs up one database's objects and data. It does not by itself back up all cluster-wide roles and tablespaces. PostgreSQL provides pg_dumpall for broader logical exports, including globals when requested.

---

## 12. MySQL logical backup with mysqldump

Run from the terminal:

```bash
mysqldump -h localhost -P 3306 -u root -p techstore > techstore_backup.sql
```

This exports the selected MySQL database as SQL text.

Restore into a new database:

First create it:

```sql
CREATE DATABASE techstore_restore_test;
```

Then run:

```bash
mysql -h localhost -P 3306 -u root -p techstore_restore_test < techstore_backup.sql
```

Validate:

```sql
USE techstore_restore_test;
SHOW TABLES;
SELECT COUNT(*) FROM customers;
```

For production, consider consistency options, storage engines, triggers, routines, events, grants and binary log/PITR requirements. A basic mysqldump command may not cover every operational requirement.

---

## 13. Connection security basics

Never:
- Commit passwords to GitHub.
- Share production credentials in class notes.
- Put passwords in screenshots.
- Use an administrator account for every pipeline.
- Disable TLS checks to hide certificate problems without understanding the risk.

Prefer:
- Dedicated service accounts.
- Least-privilege grants.
- Environment variables or a secret manager.
- TLS for network connections where required.
- Credential rotation.
- Separate development, test and production credentials.

Example environment variable names:

```text
DB_HOST
DB_PORT
DB_NAME
DB_USER
DB_PASSWORD
```

The values should be supplied securely by the environment, not committed into source control.

---

## 14. Database operations checklist

Before a pipeline runs:
- Confirm target host and database.
- Confirm the expected role is used.
- Check connection works.
- Confirm required permissions.
- Confirm the database is not in an unexpected maintenance state.
- Confirm source/target tables exist.
- Check available storage and connection capacity where relevant.

After a pipeline runs:
- Verify row counts.
- Verify key uniqueness.
- Verify required values are not NULL.
- Compare totals and reconciliation metrics.
- Record the batch ID and execution status.
- Confirm failed work was rolled back or safely quarantined.

For recovery:
- Identify the required recovery point.
- Locate a valid backup.
- Restore to an isolated target.
- Verify objects, permissions and row counts.
- Reconcile important business totals.
- Document the restore outcome.

---

## 15. Mini-project: Database Operations Checklist

### Business situation

You are a junior Data Engineer supporting the TechStore daily pipeline. The team asks you to prepare an operational checklist for database connectivity, permissions and recovery.

### Task 1: identify the environment

Run the PostgreSQL or MySQL identity/version queries. Record:
- Server version.
- Current database.
- Current user.
- Host and port used by your client.

Do not include passwords in your report.

### Task 2: inspect objects

Use psql or MySQL client commands to list databases, tables and users/roles. Identify which schema contains the training tables.

### Task 3: inspect privileges

Ask your instructor/admin to create a read-only role if your account cannot create roles. Verify that the role can SELECT the approved tables and cannot UPDATE them.

### Task 4: inspect active sessions

Use pg_stat_activity or SHOW PROCESSLIST. Identify active and idle sessions. Do not terminate sessions during the exercise.

### Task 5: create a logical backup

Create a PostgreSQL custom-format backup or MySQL SQL backup. Record the filename, timestamp, source database and exit status.

### Task 6: restore into a test database

Create a new empty restore-test database. Restore the backup there, not over the source database.

### Task 7: validate the restore

Compare:
- Table names.
- Row counts.
- Primary key counts.
- Sample records.
- Business totals where applicable.

### Task 8: document recovery

Write what you would do if the production database were accidentally damaged. Include who must approve recovery, how you select a recovery point, and how you validate the restored system.

### Project checklist

- [ ] I can identify the database and current role.
- [ ] I can explain host, port, database and username.
- [ ] I understand least privilege.
- [ ] I can distinguish PostgreSQL roles and MySQL accounts.
- [ ] I can inspect active sessions.
- [ ] I can explain logical backup and restore.
- [ ] I created a practice backup.
- [ ] I restored into a separate test database.
- [ ] I compared row counts and sample data.
- [ ] I understand why restore testing matters.
- [ ] I did not expose credentials.

---

## 16. Common beginner mistakes

1. Running a restore against the wrong database.
2. Assuming a successful backup command proves recoverability.
3. Never testing the restore process.
4. Saving credentials in a SQL file or GitHub repository.
5. Granting all privileges to a pipeline user.
6. Confusing PostgreSQL roles with MySQL user@host accounts.
7. Treating psql backslash commands as standard SQL.
8. Running pg_dump inside a SQL query editor.
9. Running pg_restore inside psql rather than the terminal.
10. Killing a database session without checking its owner and transaction.
11. Forgetting to validate restored row counts.
12. Assuming one database backup includes all cluster-level users and settings.
13. Ignoring storage, connections and operational capacity.
14. Restoring directly over production without an approved recovery plan.

---

## 17. Practice questions

1. What does a database administrator do?
2. What is the difference between a database server and a database?
3. What is a schema?
4. What is a database role?
5. What is least privilege?
6. What is the difference between PostgreSQL roles and MySQL accounts?
7. What are host and port?
8. What is an active database session?
9. What is an idle-in-transaction session?
10. What is a logical backup?
11. What is a physical backup?
12. What is the difference between backup and restore?
13. What does pg_dump do?
14. What does pg_restore do?
15. What does mysqldump do?
16. Why restore to a separate test database?
17. Why are backups not complete until restore-tested?
18. Why should credentials not be committed to GitHub?
19. How do you validate a restored database?
20. What would you check when a pipeline cannot connect?

### Answer hints

1. Availability, security, recovery, performance and operational management.
2. Software service versus a named data collection managed by it.
3. Namespace for objects.
4. Identity/permission-bearing principal.
5. Grant only required permissions.
6. PostgreSQL uses roles; MySQL accounts include user and host.
7. Server location and network port.
8. A current client connection.
9. A session with an open transaction but no active statement.
10. Export of logical objects/data.
11. Copy of storage-level database files using database-native methods.
12. Copy versus rebuilding from the copy.
13. PostgreSQL logical database export.
14. Restores a PostgreSQL archive.
15. Exports MySQL logical SQL.
16. To avoid damaging the source and prove recovery safely.
17. A backup may be incomplete, corrupt or missing required objects.
18. To prevent credential exposure.
19. Objects, row counts, keys, sample records and business totals.
20. Host, port, service, credentials, TLS, firewall, database name, role privileges and connection limits.

---

## 18. Day 28 revision

- A Data Engineer must understand database operations, not only SQL.
- Verify the database and role before changing data.
- Use least privilege for pipeline accounts.
- PostgreSQL roles and MySQL accounts have different permission models.
- Connection problems can be caused by host, port, credentials, network, TLS or privileges.
- Inspect active sessions carefully.
- A backup is not proven until a restore is tested.
- pg_dump and pg_restore are PostgreSQL logical backup tools.
- mysqldump is a common MySQL logical backup tool.
- Restore into an isolated test database for practice.
- Validate object lists, row counts and business totals.
- Never commit credentials to source control.

**You are ready for Day 29 when you can explain database connections, permissions and a safe backup/restore verification process.**
