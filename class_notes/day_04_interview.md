# Day 4 Interview Preparation: Data Architecture and Integration

**Mini-project:** ETL/ELT Architecture Designer

## 1. Core questions

### 1. What is ETL?
Extract, Transform, Load. Data is extracted from sources, transformed in a processing layer and then loaded into the target.

### 2. What is ELT?
Extract, Load, Transform. Source data is loaded into the target first and transformed using the target platform's compute capabilities.

### 3. What is a data lake?
A storage architecture for large volumes of diverse data, often in object storage. It can retain raw files and support later processing.

### 4. What is a data warehouse?
An analytical store that integrates and organizes data for reporting, business intelligence and historical analysis.

### 5. What is a lakehouse?
An architecture that combines flexible lake storage with managed table and analytical capabilities. Actual features depend on the platform and table format.

### 6. What is Kafka?
A distributed event-streaming platform used to publish, retain and consume event records through topics and partitions.

### 7. What is dbt?
A tool for managing SQL transformations in a data warehouse or lakehouse, including model dependencies, tests, documentation and lineage.

## 2. Difference-based questions

| Compare | Strong answer |
|---|---|
| ETL vs ELT | Transform before load vs load before transform |
| Lake vs warehouse | Flexible diverse storage vs curated analytical store |
| Warehouse vs lakehouse | Warehouse-centered analytical architecture vs lake storage with managed table capabilities |
| Kafka vs dbt | Event transport/streaming vs SQL transformation management |
| Ingestion vs transformation | Moving source data vs applying technical/business rules |
| Raw vs curated | Source-preserving records vs consumer-ready data |

## 3. Practical implementation questions

### 8. How would you design an ETL pipeline?
Identify source and target contracts, extract safely, validate source records, transform with documented rules, load transactionally or idempotently, reconcile counts and totals, and monitor each run.

### 9. When would you choose ELT?
When the target platform can store raw data and efficiently execute transformations, and when preserving source data for reprocessing is valuable.

### 10. Where does dbt fit?
After ingestion, when data is available in a supported warehouse/lakehouse. dbt builds SQL models and manages tests, documentation and dependencies.

### 11. Where does Kafka fit?
At event ingestion and distribution boundaries where producers publish ongoing events and multiple consumers need scalable, durable access.

## 4. Scenario questions

### Scenario A: Daily reporting and instant payment alerts
**Problem:** One system needs daily sales totals; another needs payment failures within seconds.

**Action:** Use a scheduled batch/incremental path for reporting and a streaming event path for alerts. Keep their contracts and monitoring separate.

**Reasoning:** Different latency requirements justify different processing patterns.

### Scenario B: Analysts cannot find trusted lake data
**Problem:** Object storage contains thousands of files with unclear meanings.

**Action:** Add catalog entries, owners, schemas, naming conventions, retention rules, quality checks and curated tables.

**Reasoning:** Storage alone does not create a usable data platform.

### Scenario C: SQL transformations are duplicated
**Problem:** Three dashboards independently calculate net sales.

**Action:** Create a shared, version-controlled transformation model, define the business rule, add tests and document the output.

**Reasoning:** Centralized definitions reduce inconsistent reporting.

## 5. Troubleshooting

### 12. ELT warehouse costs are increasing
Inspect repeated scans, selected columns, partition filters, incremental strategies, model materialization and schedule frequency. Optimize based on query evidence.

### 13. Kafka consumer lag grows
Inspect event arrival rate, consumer throughput, partition assignment, slow downstream writes, retries and resource limits.

### 14. A dbt model fails after a source schema change
Inspect the compiled SQL and upstream schema, update the source/model contract, test downstream dependencies and deploy a compatible change.

## 6. Follow-up questions

- What is a staging model?
- What is a data mart?
- What is a Kafka partition?
- What is consumer lag?
- What is a dbt test?
- Why is lineage useful?
- What is schema-on-read?
- How do you make a load idempotent?

## 7. Strong revision points

- Explain architecture in terms of business requirements.
- ETL and ELT are patterns, not competing brands.
- A lake, warehouse and lakehouse have different responsibilities.
- Kafka transports events; dbt manages SQL transformations.
- Every layer needs ownership, schema, quality and access expectations.
- Separate low-latency and batch requirements when necessary.

## 8. Mini-project defence

Defend the ETL/ELT Architecture Designer by explaining:

1. Which sources use batch extraction and which use events.
2. Where raw data is retained.
3. Why a warehouse or lakehouse is selected.
4. How Kafka supports payment events.
5. How dbt builds curated analytical models.
6. How failures, data quality and access are managed.

A strong answer makes trade-offs explicit and avoids claiming that one architecture fits every company.
