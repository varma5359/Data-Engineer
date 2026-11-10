# Day 4 — Data Architecture and Integration | Interview Preparation

## Core Q&A

### 1. What is ETL?
Extract data from sources, transform it in an integration/processing layer, then load prepared results into the destination.

### 2. What is ELT?
Extract and load source data into the destination first, then transform using destination compute. Raw landing is commonly retained for replay and audit.

### 3. ETL vs ELT: which is better?
Neither is universally superior. ETL may fit controlled preprocessing or destinations with limited transformation capability. ELT fits warehouse-native SQL and scalable destination compute. Consider latency, security, cost, skills, and data volume.

### 4. What is a data lake?
Flexible storage for data in many formats, often on object storage. It needs cataloging, governance, access controls, quality, and lifecycle management.

### 5. What is a data warehouse?
A curated analytical system containing integrated, historical, modeled data for reporting and analysis.

### 6. What is a lakehouse?
An architecture that combines lake storage flexibility with managed analytical table features such as schema enforcement and transaction/optimization capabilities (implementation varies by platform).

### 7. Where does Kafka fit?
Kafka transports and retains event streams through topics, producers, and consumers. It supports event-driven and streaming pipelines; it does not replace all databases, batch jobs, or orchestration.

### 8. What is dbt?
A tool/workflow for building SQL transformations as models with dependencies, tests, and documentation in an analytical data platform. It is not a general API extraction tool.

## Comparison
| Component | Main purpose |
|---|---|
| Data lake | Store flexible raw/processed data |
| Warehouse | Curated analytics and reporting |
| Lakehouse | Managed analytical tables on lake-oriented storage |
| Kafka | Event streaming transport |
| dbt | SQL transformation/model workflow |

## Scenario: choose ETL or ELT
A warehouse supports powerful SQL and the company wants raw history retained. ELT is a reasonable option: ingest raw, validate ingestion, build staged and curated SQL models, test outputs, and monitor freshness. Mention data access controls on raw data.

## Scenario: data lake becomes a data swamp
Introduce ownership, catalog/metadata, schemas/contracts, naming conventions, quality checks, retention, access policy, and discoverability. Governance is part of lake design.

## Follow-up prompts
- What is schema-on-read vs schema-on-write?
- What are staging and curated layers?
- How does dbt track model dependencies?
- How would you replay data after a transformation bug?
- How do you secure raw customer data?

## Quick revision
Explain the architecture end-to-end and justify each component based on requirements.
