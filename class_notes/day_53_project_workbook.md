# Day 53 Project Workbook — Cloud Database Design

## Goal

Design a secure managed database environment and demonstrate the connection configuration pattern. Do not create a paid database unless you have an approved account and have checked current pricing.

## Scenario

TechStore's order application stores orders in a relational database. A cloud pipeline must read approved order records. Analysts must not connect directly to the production database.

## Step 1 — Define database requirements

Document:
- database engine;
- database purpose;
- expected read/write workload;
- approximate data volume (mark estimates);
- availability requirement;
- recovery point and recovery time expectations;
- data sensitivity;
- consumers;
- environment.

## Step 2 — Draw the network path

Draw:

Pipeline compute -> VPC/subnet route -> Database security group -> Database endpoint -> Database authentication -> SQL permission

Explain what can fail at each step.

## Step 3 — Design access

Create a permission matrix:

| Identity | Network access | Database permission |
|---|---|---|
| Pipeline role | Approved compute path | Read required tables |
| Application role | Approved application path | Required insert/update actions |
| Analyst | No direct production access by default | Read curated reporting data |
| DBA | Controlled admin path | Administrative tasks |

Keep network access and SQL permissions separate.

## Step 4 — Plan connection configuration

Document environment variable names for:
- host;
- port;
- database;
- username;
- password/secret.

Do not put real credentials in the workbook or Git repository.

## Step 5 — Run a local connection exercise if available

If you have a local PostgreSQL database:
1. install psycopg;
2. define environment variables securely;
3. connect;
4. run SELECT COUNT(*) on a practice table;
5. close cursor and connection.

Record your actual output. If no database is available, explain the connection flow and submit the code as a reviewed template, not as executed evidence.

## Step 6 — Parameterized query review

Write a query that selects an order by order_id using a parameter placeholder. Explain why string concatenation is unsafe.

## Step 7 — Availability and recovery decision

Compare:
- automated backups;
- snapshots;
- multi-AZ/high availability;
- read replicas;
- disaster recovery.

Explain which requirement each feature addresses. Do not treat them as interchangeable.

## Step 8 — Troubleshooting matrix

For each failure, record likely layer and first check:
- DNS/endpoint resolution;
- connection timeout;
- connection refused;
- authentication failed;
- permission denied;
- TLS/certificate failure.

## Step 9 — Security checklist

Include:
- private access where appropriate;
- restricted security group;
- approved secret management;
- TLS;
- least-privilege database grants;
- no secrets in source control;
- backup restore testing;
- cost cleanup.

## Step 10 — Submit

Submit:
1. requirements document;
2. architecture/network diagram;
3. access matrix;
4. connection configuration example;
5. parameterized SQL example;
6. availability/recovery decision;
7. troubleshooting matrix;
8. security checklist.

## Completion criteria

A reviewer should understand how the pipeline reaches the database, how it authenticates, what data it can read, how credentials are protected and how recovery is planned.
