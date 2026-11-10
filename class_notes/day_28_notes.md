# Day 28: Database Administration Fundamentals

**Pathway:** TechPhile.ai — Data Engineer Career Pathway  
**Phase:** 3 — SQL and Database Engineering  
**Audience:** Freshers, college students and junior Data Engineer candidates  
**Level:** Beginner to junior | **Class time:** 2–3 hours  
**Approved syllabus:** MySQL and PostgreSQL administration, users, roles, permissions, backup, restore, connection management  
**Mini-project:** Database Operations Checklist  
**Primary database:** PostgreSQL; MySQL differences are identified where relevant.

---

## 1. How to study

Read one concept, predict what the query or program will do, execute the small example, compare the result and explain it aloud. Do not run the entire lesson as one large script.

## 2. Prerequisite and learning outcomes

Use the TechStore training database from Day 20: customers, products, orders and order_items. Confirm you are connected to the correct practice database. If sample data differs, inspect it before comparing outputs. Never experiment on production data.

By the end, you should understand each item in today's syllabus, execute the main examples, explain the output, recognize common mistakes and complete **Database Operations Checklist**.

## 3. Database administration

Administration keeps a database available, secure, recoverable and manageable. A Data Engineer should understand environment, permissions, backup/restore, connection troubleshooting and safe operations.

## 4. PostgreSQL and MySQL

| Area | PostgreSQL | MySQL |
|---|---|---|
| Client | psql, pgAdmin | mysql client, Workbench |
| Identity | Roles; LOGIN attribute | User accounts |
| Backup | pg_dump, pg_restore | mysqldump, MySQL Shell |
| Common default port | 5432 | 3306 |

Defaults may be changed. Confirm actual settings.

## 5. PostgreSQL roles and least privilege

A role can own objects and receive privileges. LOGIN permits connecting.

```sql
CREATE ROLE techstore_reader LOGIN;
GRANT CONNECT ON DATABASE techstore_training TO techstore_reader;
GRANT USAGE ON SCHEMA public TO techstore_reader;
GRANT SELECT ON public.customers,public.products,
 public.orders,public.order_items TO techstore_reader;
```
Do not place real credentials in shared scripts or Git. Provision secrets securely.

Verify:
```sql
SELECT has_database_privilege('techstore_reader',
 'techstore_training','CONNECT'),
 has_table_privilege('techstore_reader',
 'public.customers','SELECT');
```
Expected true/true when grants and role are correct.

Revoke a direct grant:
```sql
REVOKE SELECT ON public.customers FROM techstore_reader;
```
Membership and PUBLIC grants can still confer access; inspect effective privileges.

Least privilege: grant only what task needs. Extractor may need SELECT; loader may need INSERT on staging. Routine ETL should not use superuser.

## 6. MySQL user example

Run only in an authorized MySQL training server:
```sql
CREATE USER 'techstore_reader'@'localhost'
IDENTIFIED BY 'SET_SECURELY_OUTSIDE_SHARED_SCRIPT';
GRANT SELECT ON techstore_training.customers
TO 'techstore_reader'@'localhost';
SHOW GRANTS FOR 'techstore_reader'@'localhost';
```
Placeholder is not a credential. Use secret manager.

## 7. Backup concepts

Logical backup exports objects/data logically; physical backup captures database storage through supported process. Full backup establishes baseline; incremental/differential capture changes since defined baseline. Point-in-time recovery restores a base backup plus transaction logs/WAL to a target time. A backup is not proven until restore is tested.

## 8. PostgreSQL backup and restore

Run in terminal with PostgreSQL tools:
```bash
pg_dump -h localhost -p 5432 -U postgres -d techstore_training -F c -f techstore_training.backup
pg_restore -l techstore_training.backup
createdb -h localhost -p 5432 -U postgres techstore_restore_test
pg_restore -h localhost -p 5432 -U postgres -d techstore_restore_test techstore_training.backup
```
pg_dump exports; -h host; -p port; -U role; -d database; -F c custom archive; -f output. Authentication may prompt. Never place password in shell history. Restore into a new empty test database. Verify objects, row counts, relationships and sample queries.

## 9. MySQL backup and restore

```bash
mysqldump -h localhost -P 3306 -u root -p techstore_training > techstore_training.sql
mysql -h localhost -P 3306 -u root -p techstore_restore_test < techstore_training.sql
```
-p prompts for password. Create an empty restore database first. Verify restored data. Follow the database/service's approved backup method.

## 10. Recovery objectives

RPO (Recovery Point Objective) is tolerable data loss measured in time. RTO (Recovery Time Objective) is target time to restore service. Example RPO 15 minutes: target no more than 15 minutes of data loss; RTO 2 hours: restore service within two hours. These are targets, not guarantees.

## 11. Connection management and pooling

Connection settings: host, port, database, username, authentication, SSL/TLS, network route, timeout and pool size. Common failures: wrong host/port, stopped server, firewall, bad credentials, missing CONNECT, wrong database, TLS mismatch or connection limit.

Connection pools reuse a bounded set of sessions, reducing setup overhead and connection storms. Bigger is not always better; consider database limits, app instances, concurrency and transaction duration. Never log secrets.

## 12. Mini-project — Database Operations Checklist

Write a runbook with reader role/grants, permission verification, PostgreSQL backup/restore commands, restore verification, credential-free connection template, sample RPO/RTO, troubleshooting steps and evidence of a tested restore.

**Deliverables:** runbook, SQL grant script, backup/restore command file, restore evidence.

## 13. Self-check

Explain role, least privilege, GRANT/REVOKE, logical/physical backup, pg_dump/pg_restore, mysqldump, restore testing, PITR, RPO/RTO and connection pooling.

## 99. Next lesson

Next: Day 29 — Python and Database Integration.