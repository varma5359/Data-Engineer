# Day 1 — Introduction to Data Engineering | Interview Preparation

## Core questions and answers

### 1. What is Data Engineering?
Data Engineering is the work of designing, building, and maintaining systems that ingest, store, transform, validate, and deliver data reliably for analytics and applications.

### 2. What does a Data Engineer do daily?
Typical work includes understanding requirements, extracting data, writing SQL/Python transformations, modeling datasets, testing quality, reviewing pipeline failures, improving performance, managing access, and documenting data.

### 3. Why do companies need Data Engineers?
Business data is distributed across databases, APIs, files, and applications. Engineers make it consistent, accessible, secure, and dependable so teams can make decisions without manually combining data.

### 4. Data Engineer vs Data Analyst?
A Data Engineer builds and maintains the data pipelines and datasets. A Data Analyst queries trusted datasets, calculates business metrics, and communicates insights through reports or dashboards. They collaborate closely.

### 5. Data Engineer vs Data Scientist?
The Engineer supplies reliable data infrastructure and datasets. The Scientist uses data for statistical analysis, experimentation, and predictive modeling. Responsibilities overlap in some organizations.

### 6. Explain a basic data pipeline.
`Source → Extract → Raw Storage → Validate/Transform → Curated Table → Consumer`
The pipeline should also log execution, track counts/freshness, handle errors, protect credentials, and support recovery.

### 7. What is raw data?
Data preserved near its original source form. It supports audit, replay, debugging, and comparison. Raw does not automatically mean unrestricted access.

### 8. What is curated data?
Data cleaned, validated, documented, and modeled for a defined use. Consumers should understand its grain, fields, business rules, and freshness.

### 9. Is Data Engineering only ETL?
No. It includes architecture, ingestion, storage, transformations, modeling, orchestration, testing, data quality, security, monitoring, performance, recovery, and governance.

### 10. What makes a pipeline production-ready?
Correctness, repeatability, idempotency where needed, validation, retries/recovery, observability, security, performance, documentation, and clear ownership.

## Comparisons
| Concept | Difference |
|---|---|
| Source vs destination | Original system vs system receiving data |
| Raw vs curated | Source-preserving vs consumption-ready |
| ETL vs ELT | Transform before loading vs load then transform |
| Batch vs streaming | Periodic bounded runs vs continuous/event-driven processing |
| Data movement vs data quality | Moving records vs proving records meet expectations |

## Scenario questions

### Scenario: Dashboard numbers differ from the source application.
A strong answer:
1. Confirm metric definition and time window.
2. Check source query and timezone.
3. Compare source/destination row counts and aggregates.
4. Inspect filters, joins, duplicates, and late records.
5. Review pipeline run and transformation version.
6. Correct the issue, backfill safely, and add a regression check.

### Scenario: A daily pipeline fails at 8 AM.
- Inspect error and affected task.
- Determine transient vs data/code/configuration failure.
- Retry only when safe.
- Check whether partial output was committed.
- Resume/reprocess from a safe checkpoint.
- Validate output and communicate impact.
- Add prevention or alert improvements.

### Scenario: A business asks for “real-time sales.”
Clarify required latency (seconds, minutes, or hourly), volume, consumers, correctness/late-event behavior, and cost. Do not choose streaming just because the word real-time is used.

## Follow-up questions
- What is idempotency and why does it matter?
- How do you handle schema changes?
- How do you detect stale data?
- What is data lineage?
- What is the difference between orchestration and scheduling?
- How do you prevent unauthorized access to raw data?

## Quick interview revision
- Explain one architecture from source to dashboard.
- Name at least five Data Engineer responsibilities beyond coding.
- Mention reliability, data quality, security, and monitoring naturally.
- Start scenario answers with business definition and evidence, not guesses.
