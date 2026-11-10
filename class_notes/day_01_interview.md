# Day 1 Interview Preparation: Introduction to Data Engineering

**Mini-project:** Data Engineering Architecture Map

## 1. Core interview questions

### 1. What is data engineering?
Data engineering is the work of designing, building, testing and operating systems that collect, transform, store and deliver reliable data for analytics and applications.

### 2. What does a Data Engineer do?
A Data Engineer connects data sources, builds ingestion and transformation pipelines, organizes data for consumption, validates quality, monitors jobs and supports recovery when failures occur.

### 3. Why do organizations need Data Engineers?
Business data is distributed across databases, APIs, files and applications. Data Engineers make it consistently available and trustworthy so teams can make decisions without manually combining disconnected sources.

### 4. What is a data pipeline?
A data pipeline is a repeatable workflow that moves data through steps such as extraction, validation, transformation and loading. It should define dependencies, failure handling and monitoring.

### 5. What is the difference between a database and a data warehouse?
An operational database supports application transactions such as placing orders. A warehouse organizes integrated, historical data for analytical queries and reporting. Some database products can serve both roles, but the workloads and design goals differ.

### 6. What is raw data?
Raw data is data retained close to the form in which it was received from a source. It provides traceability and supports reprocessing when transformation logic changes.

### 7. What is data quality?
Data quality is the degree to which data meets defined expectations. Common dimensions include completeness, validity, consistency, uniqueness, accuracy and timeliness.

### 8. What is observability?
Observability is the ability to understand a pipeline's behavior from outputs such as logs, metrics, traces, run status and alerts.

## 2. Role comparison questions

### 9. Data Engineer vs Data Analyst?
A Data Engineer builds reliable data systems and datasets. A Data Analyst uses data to answer business questions through SQL, analysis and dashboards. They collaborate closely.

### 10. Data Engineer vs Data Scientist?
A Data Engineer builds data foundations and pipelines. A Data Scientist develops statistical or machine-learning solutions. Data Scientists depend on reliable, well-understood data.

### 11. Does a Data Engineer need business knowledge?
Yes. Business definitions determine how data should be interpreted. For example, gross order value, paid revenue and refunded revenue are not interchangeable.

## 3. Difference-based questions

| Compare | Strong distinction |
|---|---|
| Batch vs streaming | Batch processes bounded groups on a schedule; streaming processes ongoing events |
| ETL vs ELT | ETL transforms before loading; ELT loads first and transforms in the destination |
| OLTP vs OLAP | OLTP supports operational transactions; OLAP supports analytical workloads |
| Raw vs curated | Raw preserves received data; curated data has documented transformations and quality rules |
| Full vs incremental load | Full reads a complete dataset; incremental reads new or changed data |

## 4. Practical implementation questions

### 12. How would you design a daily order pipeline?
1. Confirm source table, keys, update behavior and business meaning.
2. Extract records using a safe full or incremental strategy.
3. Store the received batch with a run identifier.
4. Validate required fields, types, uniqueness and row counts.
5. Apply documented business transformations.
6. Load the warehouse using a duplicate-safe strategy.
7. Compare source and target metrics.
8. Record run status and alert on failure.

### 13. How would you check whether a pipeline succeeded?
I would check the run status, start and end times, processed row counts, rejected rows, source-to-target reconciliation, freshness and error details. A green scheduler status alone may not prove that the data is correct.

### 14. What would you do if a dashboard showed lower revenue than the source?
I would first confirm the revenue definition and time window. Then I would trace records through extraction, filtering, joins and loading; compare counts and totals at each stage; inspect late, rejected, duplicated or missing records; and document the root cause.

## 5. Scenario questions

### Scenario A: The same file arrives twice
**Problem:** A daily sales CSV is delivered twice and the pipeline processes both copies.

**Action:** Identify the file or batch using a stable source identifier, retain ingestion metadata, and make the target load idempotent. Reconcile the resulting record count and amount.

**Reasoning:** A retry or duplicate delivery must not double business totals.

### Scenario B: A payment API is unavailable
**Problem:** The payment endpoint times out.

**Action:** Apply bounded retries with backoff for transient failures, record the failed request context without exposing secrets, alert if the retry policy is exhausted, and resume from a safe checkpoint.

**Reasoning:** Temporary network failures should not require manual data reconstruction.

### Scenario C: A new source column appears
**Problem:** The source adds `payment_method`.

**Action:** Confirm its meaning and type, assess downstream impact, update the schema contract and transformations, test compatibility, then deploy through the normal change process.

**Reasoning:** Schema changes can break consumers even when the source considers them backward-compatible.

## 6. Troubleshooting questions

### 15. A scheduled job did not run. What do you check?
- Scheduler state and dependency status.
- Previous task failure or missed schedule.
- Credentials and network connectivity.
- Resource availability.
- Recent deployments or configuration changes.
- Logs and alert history.

### 16. Row counts match but totals do not. Why?
Rows may contain different values, duplicates may be offset by missing rows, joins may multiply amounts, or filters may differ. Compare keys and measures, not only counts.

### 17. Why should you avoid silently deleting invalid records?
It removes evidence and can hide source problems. Quarantine rejected records, record the reason and owner, and define an approved correction or exclusion policy.

## 7. Follow-up questions interviewers may ask

- How would you handle late-arriving records?
- How do you prevent duplicate processing?
- What is a checkpoint?
- What should a pipeline log?
- How do you protect personal data?
- What is the difference between a retry and a backfill?
- How would you estimate pipeline cost?
- Who owns a data definition?

A strong candidate explains trade-offs and verification, not just tool names.

## 8. Strong revision points

- Start with the business need, not the technology.
- Explain the complete source-to-consumer flow.
- Separate raw preservation from curated transformation.
- Include quality checks, monitoring and recovery in every design.
- State assumptions explicitly.
- Never claim a pipeline is reliable without explaining how duplicate processing and failures are handled.

## 9. Mini-project defence: Data Engineering Architecture Map

Be ready to explain:

1. Why each source uses its selected ingestion method.
2. Why raw data is retained.
3. Where validation occurs.
4. Which business rules are applied.
5. How the warehouse is loaded safely.
6. How failed runs are detected and recovered.
7. How sensitive fields are protected.

**Interview closing statement:** “I would design the pipeline so that every run is traceable, validated and safe to retry. I would also make the business rules and recovery process clear to the next engineer.”
