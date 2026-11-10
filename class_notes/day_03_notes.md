# Day 3: Data Sources and Formats

**Phase:** 1 — Data Engineering Foundations  
**Learning time:** 2–3 hours  
**Mini-project:** Data Source Inventory

## 1. Learning outcomes

- Identify common enterprise data sources.
- Distinguish structured, semi-structured and unstructured data.
- Understand CSV, JSON, XML and Parquet at a practical level.
- Select an ingestion approach based on source characteristics.
- Build a source inventory that records ownership, schema, volume and refresh behavior.

## 2. What is a data source?

A data source is the system, application, device or file location where data originates.

Examples:

- A customer database.
- A payment provider's REST API.
- A daily sales CSV file.
- Application logs.
- Website click events.
- Images uploaded by customers.
- IoT temperature readings.

Before building a pipeline, understand the source's owner, access method, schema, update pattern and reliability.

## 3. Structured data

Structured data follows a defined schema, often rows and columns.

Example customer table:

| customer_id | name | city | created_at |
|---|---|---|---|
| 1001 | Asha | Hyderabad | 2026-01-05 |
| 1002 | Kiran | Pune | 2026-01-08 |

Characteristics:

- Fields have defined names.
- Data types can be specified.
- Records follow a consistent shape.
- Relational databases are a common storage system.

Structured data is easier to query consistently, but it can still contain invalid or misleading values.

## 4. Semi-structured data

Semi-structured data does not always use a fixed row-and-column layout, but it contains labels, keys or markers that describe its structure.

Examples:

- JSON API responses.
- XML documents.
- Application event messages.
- Nested records.

Example JSON:

```json
{
  "order_id": 501,
  "customer": {
    "customer_id": 1001,
    "city": "Hyderabad"
  },
  "items": [
    {"product_id": "P10", "quantity": 2}
  ]
}
```

The order contains nested customer and item information. A pipeline may need to flatten or normalize nested structures for analytical use.

## 5. Unstructured data

Unstructured data does not naturally follow a fixed tabular schema.

Examples:

- Product images.
- Audio recordings.
- Videos.
- PDF documents.
- Free-form email text.

A data platform may store the original objects in object storage and maintain metadata in a database or catalog.

Do not assume that unstructured means useless or impossible to process. Specialized tools can extract text, metadata, images or features.

## 6. Compare the three categories

| Category | Structure | Examples | Common handling |
|---|---|---|---|
| Structured | Fixed fields and types | Relational tables | SQL and schema validation |
| Semi-structured | Keys, tags or nested fields | JSON, XML | Parse and validate schema |
| Unstructured | No consistent tabular structure | Images, audio, documents | Object storage and specialized processing |

A single business workflow can contain all three categories.

## 7. Databases as sources

A relational database stores data in related tables.

Example:

- `customers` table: customer attributes.
- `orders` table: order header information.
- `order_items` table: products and quantities.
- `payments` table: payment status and amount.

Relationships are commonly represented through primary and foreign keys.

Before extracting, identify:

- Primary key.
- Update timestamp or change-tracking column.
- Expected row volume.
- Transaction isolation and source load constraints.
- Whether deletes are physically removed or represented by a status.
- Whether the source offers a replica or export mechanism.

## 8. APIs as sources

An API exposes data or functionality through a defined interface.

A REST API commonly uses HTTP methods:

| Method | Typical purpose |
|---|---|
| GET | Retrieve data |
| POST | Create a resource or submit a request |
| PUT | Replace a resource |
| PATCH | Partially update a resource |
| DELETE | Delete a resource |

An API response may be JSON. The extractor must handle authentication, pagination, rate limits, timeouts, retries and changing schemas.

Never assume that one GET request returns every record. Many APIs paginate results.

## 9. Files as sources

Common files include:

- CSV: delimited text rows.
- JSON: key-value and nested structures.
- XML: tag-based hierarchical structure.
- Parquet: columnar binary format designed for analytical workloads.
- Excel: workbook with sheets, formatting and formulas.

Files may arrive through email, SFTP, shared storage, cloud buckets or manual uploads.

A production ingestion process should verify file name, arrival time, size, encoding, schema and completeness before processing.

## 10. CSV format

CSV stores tabular values as text separated by delimiters.

Example:

```csv
order_id,customer_id,amount,status
501,1001,1250.00,PAID
502,1002,800.00,PENDING
```

Important considerations:

- Delimiter may be comma, tab or semicolon.
- Text values may contain delimiters and require quoting.
- Encoding can affect special characters.
- Empty values need a defined interpretation.
- Type conversion is usually required.
- CSV does not reliably store rich data types.

A column containing `00125` may be an identifier, not a number. Converting it to an integer would remove leading zeros.

## 11. JSON format

JSON supports objects, arrays, strings, numbers, booleans and null values.

Example:

```json
{
  "order_id": 501,
  "status": "PAID",
  "items": [
    {"sku": "P10", "qty": 2},
    {"sku": "P11", "qty": 1}
  ]
}
```

Important considerations:

- Fields may be nested.
- Arrays can contain repeated child records.
- Optional fields may be absent.
- Null and missing are not always equivalent.
- Types can vary if producers are inconsistent.

For analytics, decide whether nested arrays should become child tables or be expanded into rows.

## 12. XML format

XML represents data using nested tags and attributes.

Example:

```xml
<order id="501">
  <status>PAID</status>
  <customer id="1001" />
  <item sku="P10">
    <quantity>2</quantity>
  </item>
</order>
```

Important considerations:

- Elements can be nested.
- Attributes and element text are different.
- Namespaces may be present.
- XML schemas can describe expected structure.
- Large XML documents may require streaming parsers.

## 13. Parquet format

Parquet is a columnar storage format commonly used in analytical data platforms.

Instead of storing each complete row together, columnar storage organizes values by column within row groups.

Why it is useful:

- Analytical queries often read only a subset of columns.
- Compression can be efficient for similar values.
- Column statistics can help skip irrelevant data.
- It works well with distributed processing engines such as Spark.

Trade-offs:

- It is not designed for manual editing in a text editor.
- Small files can create overhead in distributed systems.
- Schema compatibility and partition design still matter.

## 14. CSV vs JSON vs XML vs Parquet

| Format | Shape | Human-readable | Typical use |
|---|---|---|---|
| CSV | Rows and columns | Yes | Simple tabular exchange |
| JSON | Objects and arrays | Yes | APIs and events |
| XML | Nested tags | Yes | Enterprise integrations and documents |
| Parquet | Columnar binary | No, not directly | Analytics and large-scale storage |

Choose based on producer requirements, consumer needs, schema complexity, file size and query patterns.

## 15. Source discovery checklist

For each source, collect:

| Field | Question |
|---|---|
| Source name | Which system creates the data? |
| Business owner | Who defines its meaning? |
| Technical owner | Who supports access and availability? |
| Access method | SQL, API, file, event or object store? |
| Format | Table, CSV, JSON, XML, Parquet, media? |
| Schema | What fields and types exist? |
| Primary key | How is a record identified? |
| Volume | How many rows/events/files arrive? |
| Frequency | How often does data change? |
| Latency | How quickly must it be available? |
| Change behavior | Can records update or be deleted? |
| Sensitivity | Does it contain personal or restricted data? |
| Retention | How long may data be stored? |
| Quality | What checks define an acceptable record? |

## 16. Mini-project: Data Source Inventory

### Scenario

A retail company wants a central analytics platform.

Sources:

1. PostgreSQL order database.
2. Payment provider API.
3. Inventory CSV from suppliers.
4. Website clickstream JSON.
5. Product image files.

### Step 1 — Inventory each source

| Source | Category | Format | Access | Expected challenge |
|---|---|---|---|---|
| Orders | Structured | Relational | SQL | Incremental changes |
| Payments | Semi-structured | JSON | REST API | Pagination and rate limits |
| Inventory | Structured | CSV | File transfer | Missing or late files |
| Clickstream | Semi-structured | JSON events | Event stream | High volume and event order |
| Images | Unstructured | Image objects | Object storage | Metadata and access control |

### Step 2 — Define record identity

Examples:

- Order: `order_id`.
- Payment: `payment_id`.
- Inventory snapshot: `supplier_id + sku + snapshot_date`.
- Click event: `event_id`.
- Product image: `product_id + object_version`.

A key must be stable enough for deduplication and reconciliation.

### Step 3 — Define schema checks

- Required identifier is present.
- Timestamp can be parsed.
- Amount is numeric and within a sensible range.
- Status belongs to an allowed set.
- Nested JSON contains required fields.
- CSV headers match the agreed contract.
- Image object has an allowed type and expected metadata.

### Step 4 — Identify security and governance needs

- Classify customer and payment-related fields.
- Avoid storing secrets in source code.
- Restrict access by role.
- Define retention and deletion requirements.
- Document the source owner and permitted use.

### Step 5 — Deliverable

Create a source inventory spreadsheet or Markdown table containing all checklist fields, assumptions, owners, risks and proposed ingestion patterns.

## 17. Common mistakes

| Mistake | Better approach |
|---|---|
| Treating every CSV field as numeric | Preserve identifiers and convert types deliberately |
| Assuming JSON has a fixed shape | Validate optional and nested fields |
| Ignoring XML namespaces | Inspect the actual document structure |
| Opening Parquet as plain text | Use a Parquet-aware tool |
| Ignoring file encoding | Detect and standardize encoding |
| Assuming a primary key exists | Confirm record identity with the source owner |
| Ignoring deletes and updates | Agree on change-capture behavior |
| Storing sensitive files in a public bucket | Apply least-privilege access |

## 18. Day 3 revision

- Sources include databases, APIs, files, events and media.
- Structured data has a defined tabular schema.
- Semi-structured data uses keys, tags or nested structures.
- Unstructured data includes images, audio and documents.
- CSV is simple tabular text; JSON and XML support nested data; Parquet is columnar.
- A source inventory prevents hidden assumptions during pipeline design.

## 19. Check your understanding

1. Is JSON structured or semi-structured? Explain.
2. Why can a CSV identifier lose information during numeric conversion?
3. Why is Parquet useful for analytical workloads?
4. What must be checked before extracting from an API?
5. Why should every source have a business owner?
6. How would you identify duplicate click events?
