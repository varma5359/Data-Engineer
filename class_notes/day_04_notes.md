# Day 4: Data Architecture and Integration

**Phase:** 1 — Data Engineering Foundations  
**Learning time:** 2–3 hours  
**Mini-project:** ETL/ELT Architecture Designer

## 1. Learning outcomes

- Explain ETL and ELT and when each is useful.
- Distinguish data lakes, warehouses and lakehouses.
- Understand the purpose of a data pipeline.
- Recognize where Kafka and dbt fit in a modern architecture.
- Design a simple integration flow and explain its trade-offs.

## 2. Why data architecture matters

A company may have orders in PostgreSQL, payments in an API, inventory in CSV files and website activity in event messages.

Without an integration architecture:

- Teams create separate extracts.
- Business rules are duplicated.
- Reports disagree.
- Source systems receive unnecessary load.
- Data lineage becomes difficult to trace.
- Failures are hard to investigate.

A data architecture defines how data is collected, stored, transformed, protected and delivered.

## 3. What is ETL?

ETL means **Extract, Transform, Load**.

1. Extract data from source systems.
2. Transform it in a processing environment.
3. Load the transformed result into the target.

Flow:

`Sources -> Extract -> Transform -> Load -> Warehouse`

Example: extract order records, standardize currencies and remove invalid rows in a processing job, then load the curated sales table into the warehouse.

ETL is useful when data must be transformed before entering the target, when the target has limited transformation capability, or when sensitive fields must be filtered before loading.

## 4. What is ELT?

ELT means **Extract, Load, Transform**.

1. Extract source data.
2. Load it into a target platform, often a cloud warehouse or lakehouse.
3. Transform it using the target's processing capabilities.

Flow:

`Sources -> Extract -> Load raw data -> Transform in target -> Curated tables`

Example: load source order records into a raw warehouse schema, then use SQL transformations to create cleaned order and sales tables.

ELT is useful when the target can scale compute, preserve raw data and run transformations close to the stored data.

## 5. ETL vs ELT

| Dimension | ETL | ELT |
|---|---|---|
| Order | Transform before load | Load before transform |
| Transformation location | Separate processing layer | Target platform |
| Raw data retention | Depends on design | Commonly retained in raw/landing layer |
| Compute | ETL engine or application | Warehouse/lakehouse |
| Useful when | Pre-load rules or target constraints matter | Target has scalable processing |
| Risk | Transformation can become a bottleneck | Raw layer can become messy without governance |

Neither approach is universally better. Many platforms combine both patterns.

## 6. What is a data lake?

A data lake stores large volumes of data in its original or lightly processed form, often in object storage.

It can hold:

- CSV and JSON files.
- Parquet datasets.
- Application logs.
- Images and documents.
- Historical snapshots.

Strengths:

- Flexible storage for varied formats.
- Separation of storage and compute in many cloud designs.
- Useful for retaining raw and historical data.

Risks:

- Poorly governed lakes can become difficult to discover and trust.
- Inconsistent schemas can break consumers.
- Small files and poor partitioning can hurt performance.
- Access control and retention need deliberate design.

A data lake is not automatically a warehouse. The organization must add cataloging, quality, access rules and curated datasets.

## 7. What is a data warehouse?

A data warehouse is an analytical data store that integrates data and organizes it for reporting and analysis.

Typical characteristics:

- Consistent business definitions.
- Historical data.
- Curated tables.
- Analytical query patterns.
- Controlled access and governance.

Example subject areas:

- Sales.
- Customers.
- Products.
- Inventory.
- Payments.

A warehouse can contain fact and dimension tables. Detailed modeling is covered later in the pathway.

## 8. What is a lakehouse?

A lakehouse combines aspects of data lakes and warehouses: flexible, lower-cost object storage with table management and analytical capabilities such as schema enforcement, transactions and performance optimization.

The exact capabilities depend on the platform and table format.

A lakehouse is not simply a folder of Parquet files. Reliable table management, metadata, concurrency and governance are important.

## 9. Compare lake, warehouse and lakehouse

| Dimension | Data lake | Data warehouse | Lakehouse |
|---|---|---|---|
| Main purpose | Store diverse data | Trusted analytics | Lake flexibility with managed analytical tables |
| Data shape | Raw to curated, varied formats | Structured and modeled | Structured and varied |
| Schema approach | Often schema-on-read, depending on design | Curated schemas | Table schemas and metadata |
| Typical users | Engineers, data scientists | Analysts, BI teams | Engineers, analysts, data scientists |
| Main risk | Data swamp | Cost or rigidity if poorly designed | Platform complexity and governance gaps |

## 10. Data pipeline architecture

A pipeline connects the source to a consumer through controlled stages.

Example layered flow:

`Source -> Landing -> Raw -> Standardized -> Curated -> Consumer`

### Landing
Temporary or initial arrival location. It may hold original files and metadata.

### Raw
Preserves received source data with minimal alteration. Supports traceability and replay.

### Standardized
Applies technical normalization such as consistent timestamps, types and naming.

### Curated
Applies approved business rules and produces consumer-ready datasets.

### Consumer
Warehouse tables, dashboards, applications, APIs or machine-learning features.

Not every project needs every layer. Use layers when they improve traceability, quality or ownership.

## 11. Introduction to Kafka

Apache Kafka is a distributed event-streaming platform.

Core terms:

- Producer: publishes records.
- Topic: named stream of records.
- Partition: ordered subset of a topic that enables parallelism.
- Broker: Kafka server that stores and serves partitions.
- Consumer: reads records.
- Consumer group: consumers cooperating to process partitions.
- Offset: position of a record within a partition.

Example: an order service publishes `OrderCreated` events to a topic. Inventory, notifications and analytics consumers independently read the events.

Kafka is useful when multiple systems need durable event distribution and scalable consumption. It introduces operational concerns such as partition design, consumer lag, retention, schema compatibility and duplicate-safe processing.

## 12. Introduction to dbt

dbt is a transformation workflow commonly used to build and manage SQL models in a data warehouse or lakehouse.

It helps teams:

- Organize SQL transformations into models.
- Define dependencies between models.
- Add data tests.
- Document tables and columns.
- Generate lineage information.
- Review transformation changes in version control.

A simple conceptual flow:

`Raw tables -> staging models -> intermediate models -> marts`

dbt does not replace every ingestion tool or orchestration platform. It primarily manages transformations inside supported data platforms.

## 13. How Kafka and dbt fit together

They solve different problems:

| Technology | Main responsibility |
|---|---|
| Kafka | Transport and retain event streams |
| Object storage | Store files and datasets |
| Warehouse/lakehouse | Store analytical tables |
| dbt | Transform warehouse/lakehouse data using SQL |
| Airflow | Schedule and coordinate workflows |
| BI tool | Present analytical results |

A system may use Kafka for event ingestion and dbt for downstream SQL modeling. They are complementary, not substitutes.

## 14. Architecture selection example

Requirement: management needs daily sales reporting, while operations needs payment-failure alerts within seconds.

Possible design:

- Orders arrive from PostgreSQL through incremental extraction.
- Payment events arrive through an event stream.
- Raw files/events are retained in a governed landing area.
- Daily sales data is transformed into warehouse tables.
- Payment events feed a low-latency operational alert process.
- dbt models build analytical marts.
- Airflow schedules batch dependencies.
- Monitoring tracks freshness, failures and data quality.

The architecture uses more than one processing pattern because the requirements differ.

## 15. Mini-project: ETL/ELT Architecture Designer

### Scenario

Retail company sources:

- PostgreSQL orders.
- Payment API.
- Inventory CSV.
- Website events.

Business needs:

1. Daily finance reporting.
2. Payment failure alert within seconds.
3. Product performance analysis.
4. Historical source retention.

### Step 1 — Record requirements

| Requirement | Target |
|---|---|
| Finance reporting | Daily by 7 AM |
| Payment alert | Within 10 seconds |
| Product analysis | Daily refresh |
| Historical retention | Defined by data governance policy |

Targets are scenario assumptions. In a real project, validate them with stakeholders.

### Step 2 — Select architecture components

| Need | Candidate component | Reason |
|---|---|---|
| Order extraction | Incremental database connector/job | Avoid full reads each run |
| Payment events | Kafka or managed event service | Low-latency distribution |
| File landing | Object storage | Durable file retention |
| Analytical storage | Warehouse/lakehouse | Historical analytics |
| SQL transformations | dbt or SQL jobs | Versioned and tested models |
| Scheduling | Airflow | Dependency management |
| Monitoring | Logs, metrics and alerts | Operational visibility |

### Step 3 — Draw both flows

Batch analytical flow:

`Orders + Inventory -> Ingestion -> Raw storage -> SQL/dbt transforms -> Warehouse marts -> BI`

Low-latency event flow:

`Payment provider -> Event ingestion -> Stream consumer -> Payment alert`

### Step 4 — Add governance

Document:

- Source and target owners.
- Field definitions.
- Data sensitivity.
- Access roles.
- Retention and deletion policy.
- Quality rules.
- Lineage.
- Recovery and replay method.

### Step 5 — Deliverable

Submit a diagram and one-page design note explaining:

- ETL or ELT choice.
- Lake, warehouse or lakehouse role.
- Why separate batch and streaming flows exist.
- Where dbt and Kafka fit.
- How data quality, security and monitoring work.

## 16. Common mistakes

| Mistake | Better practice |
|---|---|
| Saying ETL is old and ELT is always better | Choose based on target capability and requirements |
| Calling object storage a warehouse | Explain modeling and query responsibilities |
| Treating a lakehouse as just files | Include table metadata, transactions and governance |
| Using Kafka for every file transfer | Match the tool to the need |
| Expecting dbt to extract every source | Use ingestion tools for source movement |
| Ignoring raw retention | Define replay and retention policy |
| Building layers without purpose | Give each layer a clear contract and owner |

## 17. Day 4 revision

- ETL transforms before loading; ELT loads before transforming.
- A lake stores diverse data; a warehouse organizes trusted analytics; a lakehouse combines capabilities.
- Pipeline layers improve traceability when their responsibilities are clear.
- Kafka distributes event streams.
- dbt organizes SQL transformations, tests and documentation.
- Architecture follows latency, volume, governance and cost requirements.

## 18. Check your understanding

1. When might ETL be preferable to ELT?
2. Why does a data lake need governance?
3. What makes a lakehouse different from a folder of Parquet files?
4. What is a Kafka topic?
5. What problem does dbt solve?
6. Why might one company use batch and streaming together?
