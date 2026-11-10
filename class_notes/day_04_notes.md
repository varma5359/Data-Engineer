# Day 4 — Data Architecture and Integration
**Pathway:** TechPhile.ai Data Engineer | **Phase:** Foundations | **Level:** Beginner

## Learning objectives
Explain ETL and ELT, compare data lake/warehouse/lakehouse, trace a pipeline, and understand where Kafka and dbt fit.

## 1. What is data integration?
Combining data from multiple systems into a consistent destination or view. Example: customer database + product files + sales API → retail analytics dataset.

Integration challenges:
- Different identifiers and schemas.
- Different update frequencies.
- Missing/duplicate records.
- Conflicting business definitions.
- Access restrictions and source outages.

## 2. ETL — Extract, Transform, Load
`Source → Extract → Transform → Load destination`

Transformations happen before loading into the final destination. Often useful when destination should receive only prepared data or when transformations occur in an integration layer.

Example: extract CSV, standardize dates and validate, then load clean rows into a database.

## 3. ELT — Extract, Load, Transform
`Source → Extract → Load raw data → Transform inside destination`

Raw data is loaded first; transformation uses the compute capabilities of the warehouse/lakehouse.

Example: land API data in raw tables, then use SQL/dbt models to build curated sales tables.

## ETL vs ELT
| ETL | ELT |
|---|---|
| Transform before final load | Load then transform |
| Integration engine performs transform | Destination compute performs transform |
| Useful for controlled/prepared loads | Useful for scalable warehouse-native SQL |
| Raw retention depends on design | Raw landing is commonly retained |

Neither is universally best. Choose based on source, destination, security, latency, transformation complexity, and cost.

## 4. Data lake
A data lake stores data in varied formats, often object storage, including raw and processed files. It can hold structured, semi-structured, and unstructured data.

Typical organization:
`raw/ → processed/ → curated/`
- Raw: original ingestion.
- Processed: standardized/cleaned.
- Curated: approved datasets for use.

A lake needs cataloging, access controls, quality, and lifecycle rules; otherwise it can become a “data swamp.”

## 5. Data warehouse
A data warehouse is organized for analytical querying and reporting. It typically integrates historical business data and provides modeled tables, dimensions, facts, and consistent metrics.

Example: sales fact joined with customer, product, and date dimensions.

## 6. Lakehouse
A lakehouse combines low-cost flexible lake storage with table-management and analytical capabilities commonly associated with warehouses (such as schema enforcement, transactions, and optimized querying, depending on technology).

Do not treat “lakehouse” as simply another folder; it includes table/metadata and processing capabilities.

## Lake vs warehouse vs lakehouse
| Data lake | Warehouse | Lakehouse |
|---|---|---|
| Flexible file/object storage | Curated analytical database | Lake storage with managed analytical tables |
| Many data types | Mostly structured analytical data | Structured + broader data |
| Schema may be applied at read/write depending design | Strong modeled schema | Table-level schema/management features |
| Requires governance/catalog | Strong modeling and governance | Requires governance/catalog too |

## 7. Kafka and dbt — introductory placement
### Kafka
A distributed event streaming platform. Producers publish events to topics; consumers read events. It is useful for event pipelines and streaming integration, not a replacement for every batch job or database.

### dbt
A transformation workflow/tool focused on SQL models in analytics environments. It helps organize transformations, dependencies, tests, and documentation. It is not normally the tool that extracts every source API.

## 8. Integrated retail architecture
`Customer DB + Product CSV + Sales API`
`→ Ingestion jobs → Raw lake/staging → Validation & transformations → Warehouse facts/dimensions → BI`

- Batch file and database ingestion can run on a schedule.
- API ingestion handles pagination and rate limits.
- Raw copies support replay.
- SQL/dbt models build curated tables.
- Quality tests check keys, values, and relationships.
- Monitoring reports freshness and failures.

## 9. Mini-project — ETL/ELT Architecture Designer
Draw two architectures for the same retail requirement:
1. ETL: transform in Python/integration layer before warehouse load.
2. ELT: land raw data, transform with SQL/dbt in warehouse.

For each, identify source, compute location, raw retention, quality checks, security boundary, and consumers. State why your design fits.

## 10. Common mistakes
- Believing ETL is obsolete or ELT always better.
- Calling every object-storage folder a lakehouse.
- Forgetting raw data retention and replay strategy.
- Treating Kafka as a database or a universal ingestion solution.
- Assuming dbt performs source extraction and infrastructure orchestration by itself.

## Quick revision
- ETL transforms before final load; ELT loads then transforms.
- Lake stores varied data; warehouse is modeled for analytics; lakehouse adds managed analytical table capabilities on lake storage.
- Kafka handles event streams; dbt organizes warehouse SQL transformations.
