# Day 4 Interview Preparation: Data Architecture and Integration

**Phase:** 1 — Data Engineering Foundations  
**Mini-project:** ETL/ELT Architecture Designer  
**Target level:** Fresher / Junior Data Engineer

> **How to prepare:** Do not memorize only one-line definitions. For every answer, explain the concept, give a TechMart example and tell the interviewer why the design choice matters. For architecture questions, state the requirement before naming tools.

---

## 1. Core concept questions

### 1. What is data architecture?

Data architecture is the overall design that explains where data comes from, how it is ingested, where it is stored, how it is transformed, how it is protected and how users consume it.

**Example:** TechMart receives orders from PostgreSQL, payments from an API and website events from an event stream. The architecture describes how those sources reach storage and become trusted sales reports.

**Follow-up:** Is an architecture diagram the same as a working pipeline?

No. The diagram describes the design. The pipeline is the implemented sequence of processing steps.

### 2. What is data integration?

Data integration combines data from different sources so that it can be used consistently.

**Example:** Combining orders, payments and deliveries to identify paid orders that have not yet been delivered.

### 3. Why do companies need data integration?

Business information is often distributed across separate applications. Integration gives teams a consistent view for reporting, operations and analysis.

### 4. What is ETL?

ETL means Extract, Transform, Load. Data is extracted from sources, transformed in a processing layer and then loaded into the final target.

**Example:** Read orders, validate amounts and standardize status values in Python, then load accepted records into a warehouse.

### 5. What is ELT?

ELT means Extract, Load, Transform. Data is extracted and loaded into the target platform first; the main transformations are then performed inside that platform.

**Example:** Load raw orders into warehouse tables and use SQL models to create clean sales tables.

### 6. What is the main difference between ETL and ELT?

ETL performs the main transformation before loading into the final target. ELT loads data first and performs the main transformation inside the target platform.

### 7. Is ELT always better than ETL?

No. The choice depends on target capabilities, data volume, privacy requirements, transformation complexity, cost, latency and team skills.

### 8. Can an architecture use both ETL and ELT?

Yes. A pipeline may perform security filtering or format conversion before loading and then use SQL transformations inside the warehouse. Real architectures often combine patterns.

### 9. What is a data lake?

A data lake is a storage architecture that can retain large volumes of diverse data, including structured, semi-structured and unstructured data. It often uses object storage.

### 10. What is a data warehouse?

A data warehouse is an analytical data platform that integrates and organizes data for reporting, business intelligence and historical analysis.

### 11. What is a lakehouse?

A lakehouse combines flexible data-lake storage with managed table and analytical capabilities commonly associated with warehouses. Its exact features depend on the platform and table format.

### 12. What is a data pipeline?

A data pipeline is a sequence of connected steps that moves data from sources to destinations and may validate, transform and deliver it.

### 13. What is the difference between data architecture and a data pipeline?

Architecture is the overall design of the platform and its components. A pipeline is a particular data flow implemented within that architecture.

### 14. What is batch processing?

Batch processing collects records and processes them as a group at a scheduled time or when a batch is ready.

**Example:** Extract yesterday's orders every night and calculate daily sales.

### 15. What is streaming processing?

Streaming processing handles events continuously or in ongoing small groups as they arrive.

**Example:** Process new order events to update an operations monitoring view.

---

## 2. Data lake, warehouse and lakehouse questions

### 16. What types of data can a data lake store?

It can store CSV, JSON, Parquet, logs, images, audio, video and other file-based data formats, depending on the storage and processing environment.

### 17. Why do organizations retain raw data?

Raw retention can support reprocessing, auditing, source issue investigation and future use cases. Retention must follow security, privacy and lifecycle policies.

### 18. What are Bronze, Silver and Gold layers?

They are common names for data zones:

- Bronze: raw or minimally changed data.
- Silver: cleaned and standardized data.
- Gold: curated business-ready data.

Not every organization uses these exact names.

### 19. What is a data swamp?

A data swamp is a poorly governed data lake where data is difficult to discover, understand or trust because metadata, quality, ownership and organization are weak.

### 20. What is OLAP?

OLAP means Online Analytical Processing. It supports analysis across dimensions such as time, customer, product and region.

### 21. What is the difference between OLTP and OLAP?

OLTP systems support operational transactions, such as creating an order. OLAP systems support analytical workloads, such as calculating monthly sales by region.

### 22. Why not run all analytical queries on the operational database?

Heavy analytical queries can compete with application transactions for resources. A separate analytical platform can isolate workloads and provide structures suited to reporting.

### 23. Is a data lake the same as a data warehouse?

No. A lake supports flexible storage of diverse data, often including raw data. A warehouse focuses on integrated, structured analytical data. Modern architectures may use both.

### 24. Does a lakehouse remove the need for governance?

No. A lakehouse still needs access control, ownership, metadata, quality checks, retention policies and monitoring.

---

## 3. Kafka interview questions

### 25. What is Apache Kafka?

Apache Kafka is a distributed event-streaming platform used to publish, retain and consume event records.

### 26. What problem does Kafka solve?

It helps applications exchange event streams without requiring every producer to connect directly to every consumer. Multiple consumer applications can read relevant events.

### 27. What is a Kafka producer?

A producer is an application that publishes records to Kafka topics.

### 28. What is a Kafka consumer?

A consumer is an application that reads records from Kafka topics.

### 29. What is a Kafka topic?

A topic is a named stream of related records, such as orders, payments or inventory events.

### 30. What is a Kafka partition?

A partition is a subdivision of a topic. Partitions help distribute data and support parallel consumption.

### 31. Is ordering guaranteed across all partitions?

No. Kafka guarantees ordering within a partition, not one global order across all partitions.

### 32. What is a Kafka broker?

A broker is a Kafka server that stores and serves topic data.

### 33. What is a consumer group?

A consumer group is a set of consumers that share the work of reading a topic. Different groups can independently consume the same topic.

### 34. What is an offset?

An offset identifies a record's position within a partition. Consumers use offsets to track reading progress.

### 35. Does Kafka guarantee that an event is processed exactly once by every application?

No. Delivery and processing semantics depend on producer, broker, consumer and application configuration. Consumers must design for retries, duplicate handling and safe processing.

### 36. Is Kafka a database?

Kafka is primarily an event-streaming platform, not a general replacement for a relational database or warehouse. It can retain records for configured periods and distribute them to consumers.

### 37. When would you use Kafka?

When events arrive continuously, multiple systems need to consume them, and low-latency event distribution is valuable. I would not introduce Kafka for a simple daily file load unless there is a clear requirement.

### 38. Can Kafka and a data warehouse be used together?

Yes. Kafka can distribute events, and a consumer or connector can deliver those events to storage or a warehouse for analytics.

---

## 4. dbt interview questions

### 39. What is dbt?

dbt is a tool for managing SQL-based data transformations in supported analytical platforms. It helps organize models, dependencies, tests and documentation.

### 40. What is a dbt model?

A dbt model is commonly a SQL file that defines a transformation. dbt builds the model in the target platform according to its configuration.

### 41. Does dbt usually extract data from every source system?

Not by itself in a typical ELT workflow. Data is commonly ingested into the warehouse by another process or tool, and dbt manages transformations after the data is available.

### 42. What is a model dependency?

A dependency exists when one model uses the output of another model. dbt can use these relationships to determine build order.

### 43. What is a dbt test?

A test checks whether data meets an expectation, such as a key being non-null or unique, or a foreign key matching a known record.

### 44. What is data lineage?

Data lineage shows where data came from and how it moved through source tables, transformations and final outputs.

### 45. What is the difference between Kafka and dbt?

Kafka is an event-streaming platform. dbt manages SQL transformations and related workflows in supported data platforms. They solve different problems and can coexist.

---

## 5. Difference-based questions

| Comparison | Strong interview answer |
|---|---|
| ETL vs ELT | ETL transforms before final loading; ELT loads first and transforms in the target |
| Lake vs warehouse | Lake supports flexible diverse storage; warehouse organizes integrated analytical data |
| Warehouse vs lakehouse | Warehouse is an analytical platform; lakehouse combines lake storage with managed table and analytical capabilities |
| OLTP vs OLAP | Operational transactions versus analytical workloads |
| Batch vs streaming | Scheduled groups of records versus ongoing event processing |
| Architecture vs pipeline | Overall system design versus a specific data flow |
| Kafka vs dbt | Event distribution versus SQL transformation management |
| Producer vs consumer | Publishes events versus reads events |
| Topic vs partition | Named stream versus subdivision of a stream |
| Raw vs curated data | Source-oriented retained data versus prepared business-ready data |

---

## 6. Scenario-based interview questions

### 46. Finance needs yesterday's sales report by 8 AM. Would you use Kafka?

**Strong answer:** Not automatically. A scheduled batch pipeline may satisfy the requirement. I would use Kafka only if there is a separate need for continuous event distribution or low-latency processing.

### 47. Operations needs to know about new orders within seconds. What would you consider?

I would consider event-driven streaming. The order application could publish order events, and a streaming platform such as Kafka could distribute them to consumers. I would also define event schema, duplicate handling, ordering needs, monitoring and recovery.

### 48. A company stores orders in a database, payments in an API and inventory in CSV files. How would you approach integration?

First, I would document each source, owner, format, update frequency and extraction constraints. Then I would design ingestion, raw retention, validation, transformation, curated storage and reporting. I would choose batch or streaming separately for each requirement.

### 49. The company wants to keep raw data for future analysis. What would you recommend?

A governed data lake or raw storage layer may be appropriate. I would define folder or table organization, metadata, ownership, retention, access permissions and quality checks.

### 50. Analysts have hundreds of SQL scripts and cannot understand dependencies. What could help?

A transformation management tool such as dbt may help organize SQL models, dependencies, tests, documentation and lineage.

### 51. A source API is unavailable during a pipeline run. What should the design consider?

The pipeline should report the failure, avoid publishing incomplete data as complete, preserve enough state to retry safely, and alert the responsible team. Retry limits and backoff should be appropriate to the API.

### 52. A file arrives twice. What is the risk?

Duplicate data may be loaded and inflate reports. The pipeline should identify files or records reliably and use idempotent loading or deduplication rules appropriate to the business key.

### 53. A raw source contains customer email addresses, but the dashboard only needs city-level sales. What should you consider?

Apply data minimization and access controls. Avoid exposing unnecessary personal information to downstream users. Depending on policy, sensitive fields may be masked or removed before broad analytical access.

### 54. A company has only a small daily CSV file and one report. Should it build a Kafka architecture?

Probably not. A simple scheduled batch pipeline may be cheaper and easier to operate. I would revisit the design if volume, latency or integration requirements change.

### 55. A dashboard total does not match the source system. How would you investigate?

Trace lineage from the dashboard to curated tables, transformations and raw data. Compare source and target row counts and totals, check filters and joins, inspect rejected records, and identify whether timing or business definitions differ.

---

## 7. Architecture design interview framework

When an interviewer asks you to design a data architecture, use this sequence:

1. Clarify the business requirement.
2. Identify data sources and owners.
3. Understand volume, format and update frequency.
4. Decide batch, streaming or a combination.
5. Describe ingestion.
6. Explain raw and curated storage.
7. Select ETL or ELT based on requirements.
8. Describe the transformation and consumption layers.
9. Include data quality, monitoring and failure handling.
10. Include security, governance and cost.
11. Explain trade-offs and why the design is suitable.

### Sample answer: Design a daily sales reporting architecture

TechMart stores orders in PostgreSQL and payment data in an API. Finance needs a daily report.

I would first confirm the report deadline, historical retention needs and definition of a successful sale. I would extract orders and payments on a schedule, land the source data in a controlled raw area, validate required fields and reconcile the records. If the target warehouse supports the workload, I could use an ELT pattern and manage SQL transformations with dbt. I would publish a curated daily sales model for reporting. I would also monitor source freshness, row counts, rejected records and pipeline failures, and restrict access to sensitive customer fields.

This answer explains the requirement, flow, tools and operational concerns without choosing technology blindly.

---

## 8. Mini-project discussion: ETL/ELT Architecture Designer

### 56. Explain your mini-project.

I designed a high-level architecture for TechMart to integrate orders, payments, inventory files and website events. I documented each source, selected appropriate ingestion patterns, separated raw and curated data, and explained where a warehouse, Kafka and dbt could fit.

### 57. Why did you not use streaming for every source?

Because not every source has a low-latency requirement. Hourly inventory files and daily reports can often use batch processing. Streaming should be introduced when continuous event processing provides business value.

### 58. Why did you retain raw data?

Raw retention supports investigation, reprocessing and future requirements. I would still apply access controls and retention policies.

### 59. What would you monitor?

I would monitor pipeline success and duration, source freshness, records read and written, rejected records, duplicates, schema changes and reconciliation totals.

### 60. What would you improve before production?

I would define service-level expectations, retry and recovery behavior, idempotency, access controls, encryption, data retention, alert ownership, deployment procedures and cost monitoring.

---

## 9. Interview practice: answer in your own words

For each question, try a 30–60 second answer before reading the suggested points.

1. Explain data architecture to a non-technical manager.
2. Give a real example of data integration.
3. Explain ETL using a business example.
4. Explain ELT using a warehouse example.
5. How do you decide between ETL and ELT?
6. Explain data lake, warehouse and lakehouse without using tool names.
7. Explain a data pipeline from source to dashboard.
8. When would batch be better than streaming?
9. When would streaming be justified?
10. Explain Kafka producer, topic and consumer.
11. Explain partitions and offsets.
12. Explain dbt to a junior analyst.
13. How do tests and lineage help a Data Engineer?
14. Design a daily sales report pipeline.
15. Explain how you would protect sensitive customer information.

### What interviewers are checking

- Can you explain the concept in simple language?
- Can you connect it to a real business problem?
- Do you understand the order of data movement?
- Can you distinguish tools that solve different problems?
- Do you consider quality, reliability and security?
- Can you justify why your design is appropriate?

---

## 10. Final revision sheet

| Topic | Remember this |
|---|---|
| Data integration | Combine data from different sources consistently |
| Data architecture | Overall plan for data movement, storage, transformation, security and use |
| ETL | Extract → Transform → Load |
| ELT | Extract → Load → Transform |
| Data lake | Flexible storage for diverse data |
| Data warehouse | Integrated analytical platform |
| Lakehouse | Lake storage with managed table and analytical capabilities |
| Data pipeline | Connected data-processing steps |
| Batch | Process groups of records |
| Streaming | Process ongoing events |
| Kafka | Publish, retain and consume event streams |
| Producer | Publishes records |
| Consumer | Reads records |
| Topic | Named stream |
| Partition | Subdivision of a topic |
| Offset | Record position in a partition |
| dbt | SQL transformation models, dependencies, tests and documentation |
| Lineage | Where data came from and how it was transformed |

**Next lesson:** Day 5 — Data Pipeline Fundamentals  
**Mini-project:** Pipeline Requirements Document
