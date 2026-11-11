# Day 65 Project Workbook — Secure Data Pipeline

## Goal
Design a secure credential and access model for an Airflow pipeline that reads a database and writes curated data to object storage.

## Scenario
TechStore has development and production environments. The pipeline needs read access to source orders and write access to curated sales. It must not expose passwords in code or logs.

## Step 1 — Inventory identities and systems
List the Airflow scheduler/worker identity, source database, API, object storage, warehouse, developers and operators.

## Step 2 — Classify configuration
Separate non-secret configuration (hostnames, non-sensitive paths, flags), secrets (passwords, tokens, private keys) and workload identity (role/service identity). Explain where each belongs.

## Step 3 — Local environment-variable exercise
Create Python code that reads DB_HOST, DB_NAME and DB_PASSWORD. Check whether the password exists but never print it. Use only a classroom placeholder secret.

## Step 4 — Design an Airflow Connection
Define connection ID, provider type, host/database fields, authentication method, secrets backend and rotation owner. Do not put a real password in the workbook.

## Step 5 — Least-privilege matrix

| Identity | Resource | Allowed action |
|---|---|---|
| ETL workload | Source orders schema | Read |
| ETL workload | Curated sales prefix | Write |
| Analyst | Curated sales prefix | Read |
| Developer | Production secret | No direct access by default |

Adjust and justify every permission.

## Step 6 — Network and encryption
Document private connectivity, TLS, encryption at rest, key permissions, firewall boundaries and audit logging.

## Step 7 — Secret leak incident
Explain how to revoke/rotate, inspect access logs, remove the exposed value, review repository history, migrate to managed secrets and add scanning.

## Step 8 — Security review
Review code, DAGs, logs and repository configuration. Confirm no real credentials or sensitive records are included.

## Submission
Submit system inventory, configuration/secret classification, environment-variable code, Connection design, permission matrix, network/encryption design, leak response plan, security checklist and README.

## Completion criteria
A reviewer can identify the workload identity, its permissions, secret storage and rotation process, and how sensitive values are kept out of code and logs.
