# Day 28 Interview Preparation: Database Administration Fundamentals

**Phase:** 3 — SQL and Database Engineering  
**Level:** Beginner to junior Data Engineer  
**Mini-project:** Database Operations Checklist  
**Databases:** PostgreSQL and MySQL

---

## 1. Core questions with complete answers

### 1. What is database administration?

Database administration includes keeping databases available, secure, recoverable and performant. It includes user management, permissions, backup and restore, connection management, monitoring and operational maintenance.

### 2. Why should a Data Engineer understand database administration?

Data pipelines depend on databases being reachable, correctly permissioned and recoverable. A Data Engineer should be able to diagnose basic connection and access problems, validate pipeline permissions and understand recovery procedures.

### 3. What is the difference between a database server and a database?

The server is the running database software/service. A database is a named collection of objects and data managed by that server.

### 4. What is a schema?

A schema is a namespace that organizes database objects. In PostgreSQL, tables and views can be referenced with schema-qualified names such as public.orders.

### 5. What is a database role in PostgreSQL?

A role is a database identity or permission-bearing principal. It can have LOGIN capability and can also act as a group role whose privileges are inherited by members.

### 6. How does MySQL identify a user?

A MySQL account includes a user name and host component, such as 'report_user'@'localhost'. Different host definitions can represent different account entries and privileges.

### 7. What is least privilege?

Least privilege means giving an account only the permissions needed for its task. A read-only reporting job should not receive broad UPDATE, DELETE or administrative rights.

### 8. What is the difference between authentication and authorization?

Authentication verifies who or what is connecting. Authorization determines what that identity is allowed to do.

### 9. What is CONNECT privilege in PostgreSQL?

CONNECT allows a role to connect to a specified database. The role may still need schema and table privileges to access objects after connecting.

### 10. What is USAGE on a PostgreSQL schema?

USAGE allows a role to access objects in a schema when it also has the required object privileges. It does not by itself grant SELECT on every table.

### 11. What is a database connection?

A connection is an active client session to the database server. It uses connection details such as host, port, database, user and authentication method.

### 12. What are common default ports?

PostgreSQL commonly uses 5432 and MySQL commonly uses 3306. Administrators can configure different ports, so verify the actual environment.

### 13. What causes a connection failure?

Possible causes include wrong host, port, database name, credentials, stopped service, firewall/network restrictions, TLS certificate issues, connection limits or authentication configuration.

### 14. What is an active session?

An active session is a client connection currently executing a query. Database monitoring views can show session state and query information, subject to privileges.

### 15. What is an idle-in-transaction session?

It is a connected session with an open transaction but no statement currently running. It can hold locks and old snapshots, so applications should commit or roll back promptly.

### 16. What is a database backup?

A backup is a copy or recovery artifact used to restore data and objects after loss, corruption or an operational mistake. A backup strategy must define scope, frequency, retention, security and recovery objectives.

### 17. What is a restore?

A restore rebuilds a database or selected objects from a backup. It should be validated to confirm the restored data and objects meet the recovery requirement.

### 18. What is a logical backup?

A logical backup exports database objects and data in a logical representation such as SQL statements or an archive. It can be selective and useful for migrations or object-level recovery.

### 19. What is a physical backup?

A physical backup copies database storage files or uses a database-native physical backup process. It is commonly used for large systems and point-in-time recovery strategies.

### 20. What is pg_dump?

pg_dump is PostgreSQL's logical backup utility for a database. It can produce plain SQL or archive formats and supports selective backup options.

### 21. What is pg_restore?

pg_restore restores PostgreSQL archive formats created by tools such as pg_dump using custom or directory formats. Plain SQL dumps are commonly restored with psql.

### 22. What is mysqldump?

mysqldump is a MySQL logical backup utility that exports database objects and data as SQL statements. The restore is commonly performed using the mysql client.

### 23. Does pg_dump back up every PostgreSQL role in the cluster?

No. pg_dump backs up a database. Cluster-wide roles and tablespaces are global objects and can be exported using pg_dumpall options.

### 24. Why restore to a separate test database?

It proves the backup can be restored without damaging the source. It allows validation of tables, data, constraints and business totals in isolation.

### 25. Why is a successful backup command not enough?

The file may be incomplete, corrupt, inaccessible or missing required objects. Only a restore test demonstrates that the backup can be used for recovery.

### 26. What should be checked after a restore?

Check object names, schemas, tables, views, constraints, row counts, key uniqueness, sample records, permissions and important business totals. Compare against the source or a trusted baseline.

### 27. What is RPO?

Recovery Point Objective is the maximum acceptable amount of data loss measured in time. For example, an RPO of 15 minutes means the organization aims to recover to a point no more than 15 minutes before the failure.

### 28. What is RTO?

Recovery Time Objective is the target time to restore service after an outage. A system with an RTO of one hour should be designed and tested to recover within that target.

### 29. What is point-in-time recovery?

Point-in-time recovery restores a database to a specific time using a base backup and required log records. PostgreSQL commonly uses WAL archiving for this type of recovery.

### 30. Why should credentials not be stored in GitHub?

A repository may be shared, copied or exposed. Committed credentials can give unauthorized access. Use secret managers or secure environment configuration, restrict access and rotate credentials if exposed.

### 31. Why should a pipeline avoid using a database superuser?

A superuser has broad authority. A pipeline usually needs only limited read/write privileges. Least privilege reduces the damage if code or credentials are compromised.

### 32. What is connection pooling?

Connection pooling reuses a controlled set of database connections instead of creating a new connection for every operation. It reduces connection setup overhead and helps control database connection counts.

### 33. What can happen if too many connections are opened?

The database may reach its connection limit, consume memory and resources, and reject new clients. Applications should reuse connections or use an appropriate pool.

### 34. What is a database maintenance window?

It is an agreed period for operational work such as upgrades, maintenance or changes that may affect availability. Changes should follow approval, backup and rollback procedures.

### 35. What is the difference between a backup and replication?

A backup is a recovery copy. Replication maintains another copy of data for availability or read scaling. Replication can also replicate accidental deletion or corruption, so it does not replace backups.

---

## 2. Scenario-based interview questions

### Scenario 1: The pipeline suddenly cannot connect to PostgreSQL

**Good answer:** I would verify the host, port, database name and service availability. Then I would check network/firewall and TLS configuration, authentication, database CONNECT privilege, and whether the server has reached its connection limit. I would capture the exact error rather than immediately changing credentials or permissions.

### Scenario 2: The pipeline can connect but receives permission denied on a table

**Good answer:** Authentication succeeded, so I would inspect authorization. Check database CONNECT, schema USAGE and table privileges. Confirm the pipeline uses the intended role and grant only the required SELECT or DML privileges.

### Scenario 3: A backup file exists but restore fails

**Good answer:** File existence does not prove backup integrity. I would inspect the backup command's exit status and logs, verify format and tool version compatibility, restore into an isolated target, and investigate missing dependencies, permissions or corruption.

### Scenario 4: A developer asks for superuser credentials for a pipeline

**Good answer:** I would not share an administrator credential. I would identify the exact required operations and create or request a dedicated least-privilege service account. Administrative actions should be separated and audited.

### Scenario 5: The database has too many idle connections

**Good answer:** I would inspect connection owners and application names, check whether connections are being closed or pooled correctly, and identify idle-in-transaction sessions. I would not terminate sessions without understanding transaction and application impact.

### Scenario 6: A database was accidentally deleted

**Good answer:** I would follow the incident and recovery procedure, identify the required recovery point based on RPO, restore to an isolated environment, apply required logs if point-in-time recovery is available, validate data and permissions, then obtain approval before redirecting production traffic.

### Scenario 7: Replication is enabled, so the team wants to stop backups

**Good answer:** Replication is not a replacement for backup. Deletes and corrupt changes may replicate. Maintain tested backups and a recovery plan.

---

## 3. Mini-project defence

Be ready to explain:

1. Which database and role did you connect with?
2. How did you verify the environment?
3. What permissions did the pipeline user require?
4. How did you apply least privilege?
5. How did you inspect active sessions?
6. Which backup utility did you use?
7. What backup format did you create?
8. Where did you restore it?
9. How did you validate row counts and business totals?
10. What would you do if the restore failed?

---

## 4. Common interview traps

- Confusing authentication with authorization.
- Granting broad privileges to a pipeline.
- Assuming CONNECT grants table access.
- Assuming a backup file is recoverable without a restore test.
- Treating replication as a backup.
- Forgetting that pg_dump does not include all cluster-wide globals.
- Running administrative commands in the wrong client.
- Exposing passwords in source code or terminal history.
- Terminating sessions without understanding locks and transactions.
- Failing to validate a restored database.

---

## 5. Readiness checklist

- [ ] I can explain server, database and schema.
- [ ] I understand PostgreSQL roles and MySQL accounts.
- [ ] I can explain least privilege.
- [ ] I can diagnose common connection failures.
- [ ] I can inspect active sessions.
- [ ] I understand logical and physical backup concepts.
- [ ] I can explain pg_dump and pg_restore.
- [ ] I can explain mysqldump.
- [ ] I understand why restore testing is mandatory.
- [ ] I can explain RPO and RTO.
- [ ] I know not to commit credentials.
