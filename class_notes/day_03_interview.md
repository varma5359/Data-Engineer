# Day 3 — Data Sources and Formats | Interview Preparation

## Core Q&A

### 1. Name common data sources.
Relational databases, REST APIs, files/object storage, logs, message brokers, SaaS platforms, and IoT devices.

### 2. Structured vs semi-structured vs unstructured?
Structured data follows a predefined tabular schema. Semi-structured data uses labels/nesting but allows variation (JSON/XML). Unstructured data includes images, audio, video, and free text without a simple row/column model.

### 3. CSV vs JSON?
CSV is row/column text and suits simple flat exports. JSON supports nested objects and arrays and is common in APIs. CSV has weak type/schema metadata; JSON can have variable structures.

### 4. Why use Parquet?
Parquet is columnar, supports compression and schema metadata, and can read selected columns efficiently for analytics. It is usually more suitable than CSV for large analytical datasets.

### 5. What is a schema?
A definition of fields and their data types, often including constraints and meaning. Schema helps validate records and communicate expectations between producers and consumers.

### 6. How do you assess a new source?
Confirm ownership, access/authentication, source of truth, unique key, update pattern, volume, schema, deletion behavior, timezone, rate limits, and quality expectations.

### 7. What problems occur during file ingestion?
Wrong encoding, malformed rows, delimiter/quote issues, duplicate files, unexpected columns, type conversion errors, missing files, and partial uploads.

## Format decision matrix
| Requirement | Suggested format/pattern |
|---|---|
| Small flat exchange | CSV |
| API payload / nested data | JSON |
| Tag-based integration | XML |
| Large analytics dataset | Parquet |
| Continuous events | Broker/event format (often JSON/Avro/Protobuf depending on platform) |

## Scenario: API adds a new field
- Do not assume it is harmless or automatically mapped.
- Check schema contract and downstream consumers.
- Decide whether to ignore, capture, or map it.
- Test compatibility and update documentation.
- Monitor unexpected fields or breaking changes.

## Scenario: customer IDs lose leading zeros
Treat identifier as string, inspect source/export behavior, restore from authoritative source if already lost, and add schema validation to prevent numeric coercion.

## Follow-up questions
- What is schema evolution?
- Why is a primary key useful for ingestion?
- How do you handle source deletes?
- What is a data contract?
- Why is Parquet not ideal for manual inspection?

## Quick revision
Discuss source format based on structure, volume, query pattern, and compatibility—not just file extension.
