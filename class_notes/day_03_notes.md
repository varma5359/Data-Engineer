# Day 3 — Data Sources and Formats
**Pathway:** TechPhile.ai Data Engineer | **Phase:** Foundations | **Level:** Beginner

## Learning objectives
Identify common source systems, classify structured/semi-structured/unstructured data, and select suitable formats for ingestion and storage.

## 1. Where does company data come from?
| Source | Example | Common access |
|---|---|---|
| Relational database | MySQL orders | SQL query/connector |
| REST API | CRM customers | HTTP + authentication |
| Files | CSV sales export | File/object storage |
| Application logs | Login events | Log platform/files |
| Message broker | Order events | Consumer subscription |
| SaaS platform | Marketing data | API/export |
| IoT device | Temperature readings | Gateway/event stream |

Each source has different permissions, limits, reliability, update frequency, and schema behavior.

## 2. Data categories
### Structured
Fixed, predictable schema (columns and types). Example: relational table with order_id, amount, date.

### Semi-structured
Has organization and labels, but fields may vary. Example: JSON API response, XML document. One record may contain optional/nested fields.

### Unstructured
No simple row/column structure. Example: images, audio, video, free-form documents. Metadata can still be structured.

## 3. Common file formats
### CSV
- Text rows separated by delimiters.
- Easy to inspect and exchange.
- Schema/type information is limited.
- Delimiters, quotes, null representation, and encoding need care.

### JSON
- Key/value structure; supports arrays and nested objects.
- Common for APIs and event payloads.
- Can be nested and inconsistent across records.

### XML
- Tag-based hierarchical format.
- Common in older integrations and some enterprise systems.
- Requires parsing and handling namespaces/attributes.

### Parquet
- Columnar binary format designed for analytical storage.
- Stores schema and supports compression/efficient column reads.
- Good for large analytical datasets and data lakes.
- Not intended for manual editing like CSV.

## Format comparison
| Format | Structure | Human-readable? | Typical use |
|---|---|---|---|
| CSV | Rows/columns | Yes | Simple exports |
| JSON | Nested keys/arrays | Yes | APIs/events |
| XML | Nested tags | Yes | Enterprise integrations |
| Parquet | Columnar | No (normally) | Analytics/storage |

## 4. Schema and data types
A schema describes fields, types, and sometimes constraints.
Example:
- order_id: integer/string identifier
- order_date: date/timestamp
- amount: decimal
- status: string

Identifiers may look numeric but should remain strings if arithmetic is meaningless or leading zeros matter (e.g., postal code/customer code).

## 5. Data source assessment checklist
Before ingestion, ask:
- Who owns the source?
- What is the source of truth?
- How do we authenticate?
- Is access read-only?
- How often does data change?
- Is there a stable unique key?
- How are deletes represented?
- Can schema change without notice?
- What are rate limits/volume?
- What timezone and encoding are used?
- How will we validate completeness?

## 6. Example API JSON
```json
{
  "order_id": 501,
  "customer": {"id": "C1008", "region": "South"},
  "amount": 2499.00,
  "status": "delivered"
}
```

This is semi-structured because customer is nested and fields may be optional. When loading to a table, define how nested fields map to columns and what happens if region is missing.

## 7. Mini-project — Data Source Inventory
Create a table with at least five sources for a retail company:
- Source name and owner
- Format/technology
- Business entity
- Update frequency
- Access method
- Unique key
- Expected volume
- Data quality risks
- Destination

Then recommend an ingestion pattern for each source.

## 8. Common mistakes
- Assuming CSV automatically has correct data types.
- Treating customer IDs as numbers and dropping leading zeros.
- Assuming every JSON record has identical fields.
- Ignoring encoding, timezone, and null conventions.
- Selecting Parquet without checking downstream tool compatibility.
- Extracting data without knowing source ownership or access policy.

## Quick revision
- Structured = fixed schema; semi-structured = flexible labeled structure; unstructured = content without simple tabular schema.
- CSV is simple interchange; JSON/XML are hierarchical; Parquet is columnar analytics format.
- Source assessment includes keys, frequency, volume, schema, access, and data quality.
