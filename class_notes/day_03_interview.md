# Day 3 Interview Preparation: Data Sources and Formats

**Mini-project:** Data Source Inventory

## 1. Core questions

### 1. What is a data source?
A system or location where data originates, such as a relational database, API, file, event stream or media store.

### 2. What is structured data?
Data organized into defined fields and types, commonly stored in relational tables.

### 3. What is semi-structured data?
Data with keys, tags or nested structures but not necessarily a fixed tabular schema. JSON and XML are common examples.

### 4. What is unstructured data?
Data without a consistent tabular layout, such as images, audio, video and free-form documents.

### 5. What is CSV?
A text format that represents rows and columns using delimiters. It is easy to exchange but does not reliably preserve rich data types.

### 6. What is JSON?
A text format built from objects, arrays and primitive values. It is common in REST APIs and event messages.

### 7. What is XML?
A tag-based hierarchical format. It supports nested elements, attributes and schema definitions.

### 8. What is Parquet?
A columnar binary file format commonly used for analytical storage and processing. It supports efficient column reads and compression.

## 2. Difference-based questions

| Compare | Strong answer |
|---|---|
| Structured vs semi-structured | Fixed tabular schema vs keys/tags and flexible nesting |
| Semi-structured vs unstructured | Machine-readable structural markers vs no consistent tabular structure |
| CSV vs JSON | Tabular delimited text vs nested objects and arrays |
| JSON vs XML | Key-value/array representation vs tag-based hierarchy |
| CSV vs Parquet | Text row format vs columnar binary analytics format |
| Database vs API | Persistent queryable data store vs interface for requesting resources or actions |

## 3. Practical implementation questions

### 9. What do you inspect before ingesting a new source?
Owner, access method, authentication, schema, key, data types, volume, refresh frequency, update/delete behavior, latency, sensitivity, retention and quality expectations.

### 10. How would you ingest a paginated API?
Read the API documentation, authenticate securely, request one page, validate the response, follow the next-page token or link, handle rate limits and retries, and persist progress so the extraction can resume safely.

### 11. How would you process nested JSON for analytics?
Parse the JSON, validate required fields, decide how arrays map to relational rows, preserve parent-child identifiers, and write the normalized result with source lineage.

### 12. Why should an ID such as 00045 remain text?
It is an identifier, not a quantity. Numeric conversion removes leading zeros and can change the key.

## 4. Scenario questions

### Scenario A: Supplier sends a malformed CSV
**Problem:** The file has an unexpected delimiter and missing column.

**Action:** Quarantine the file, record the validation failure, compare it with the agreed schema, contact the supplier owner and process only after an approved correction or compatibility rule.

**Reasoning:** Guessing columns can silently corrupt downstream data.

### Scenario B: API adds a nested field
**Problem:** A response now contains `customer.address.postal_code`.

**Action:** Check whether the field is optional, update schema documentation and tests, and assess downstream consumers before deployment.

**Reasoning:** Schema evolution must be managed deliberately.

### Scenario C: A Parquet dataset has thousands of tiny files
**Problem:** Query performance is poor despite columnar storage.

**Action:** Inspect file sizes and partition design, compact small files where appropriate, and review query filters and selected columns.

**Reasoning:** A good format does not compensate for poor file layout.

## 5. Troubleshooting

### 13. CSV values appear corrupted
Check encoding, delimiter, quoting, line endings, decimal conventions and type inference.

### 14. JSON parser fails
Inspect malformed syntax, encoding, unexpected nulls, wrong data types and truncated responses. Capture a safe sample without exposing secrets or personal data.

### 15. Source and target counts differ
Check pagination, filters, late records, duplicate handling, rejected rows and the time window used for comparison.

## 6. Follow-up questions

- What is a schema contract?
- What is a primary key?
- What is a composite key?
- What is a nullable field?
- What is schema evolution?
- What is a partition column?
- Why is source lineage useful?
- How would you store an image and its metadata?

## 7. Strong revision points

- Never design ingestion from a file extension alone; inspect actual content.
- Confirm identity and update behavior with the source owner.
- Keep source identifiers stable.
- Validate schema before applying transformations.
- Preserve rejected records and failure reasons.
- Treat sensitive source data according to its classification.

## 8. Mini-project defence

Explain your inventory for the five retail sources:

1. Why each source belongs to its data category.
2. How it will be accessed.
3. What uniquely identifies a record.
4. How often it changes.
5. Which validation checks apply.
6. What security and retention rules are needed.

A strong candidate identifies unknowns and asks the source owner rather than inventing assumptions.
