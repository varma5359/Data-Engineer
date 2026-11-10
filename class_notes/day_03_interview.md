# Day 3 Interview Preparation: Data Sources and Formats

**Phase:** 1 — Data Engineering Foundations  
**Mini-project:** Data Source Inventory  
**Target level:** Fresher / Junior Data Engineer

> **Practice method:** For every answer, define the term, give one example and explain why it matters to a pipeline. For source-design questions, state what you know and what you would confirm with the source owner.

---

## 1. Core data source questions

### 1. What is a data source?

A data source is the original system, application, service, device or file location from which data is obtained. Examples include databases, APIs, CSV files, application logs and IoT sensors.

### 2. What is the difference between a source and a destination?

A source is where data originates; a destination is where data is delivered. An order database may be the source and a warehouse table may be the destination.

### 3. What is a data consumer?

A consumer is a person, application or process that uses data. Examples include a dashboard, analyst, data science workflow or operational service.

### 4. What is a field?

A field is one named piece of information, such as order_id, amount or order_status.

### 5. What is a record?

A record is a collection of related field values. In a relational table, it is commonly represented as a row.

### 6. What is a schema?

A schema describes the expected structure of data, including fields, data types, required values and sometimes relationships or constraints.

### 7. What is a data type?

A data type describes the kind of value a field is expected to contain, such as text, integer, decimal, Boolean, date or timestamp.

### 8. Why should an ID such as 00045 be stored as text?

An identifier is not necessarily a quantity. Numeric conversion can remove leading zeros and change the identifier. Preserve the exact representation defined by the source contract.

---

## 2. Data structure questions

### 9. What is structured data?

Structured data follows a defined organization, commonly rows and columns with known fields and data types. A relational customer table is an example.

### 10. What is semi-structured data?

Semi-structured data contains keys, tags or nested structures but may not follow one fixed flat table layout. JSON API responses and XML documents are common examples.

### 11. What is unstructured data?

Unstructured data does not naturally follow a consistent tabular layout. Images, audio, video and free-form documents are examples.

### 12. What is the difference between structured and semi-structured data?

Structured data commonly has a fixed tabular schema. Semi-structured data uses labels, keys or nested structures and may allow records to have different optional fields or shapes.

### 13. Is structured data always clean?

No. A table can contain missing values, invalid dates, duplicate identifiers and incorrect business values. Structure does not guarantee quality.

### 14. Can one business process contain all three data categories?

Yes. A product catalog may contain structured product IDs and prices, semi-structured attributes from an API and unstructured product images or videos.

### 15. What is nested data?

Nested data is a structure contained inside another structure. For example, an order object may contain a customer object and an array of order items.

---

## 3. Database source questions

### 16. How can a Data Engineer obtain data from a database?

Common approaches include approved SQL queries, database connectors, read replicas, scheduled exports or change data capture. The method depends on source capabilities and access approval.

### 17. What information should you collect before ingesting a database table?

Database product, approved connection details, table/view names, schema, primary or business key, data volume, update/delete behavior, refresh expectation, source owner and access permissions.

### 18. What is full extraction?

Full extraction reads the complete selected dataset. It may be suitable for smaller datasets or initial loads, but can become expensive when repeated on large datasets.

### 19. What is incremental extraction?

Incremental extraction reads new or changed records since a known point, using a timestamp, sequence, change log or other reliable mechanism. It reduces repeated work but requires careful tracking and recovery.

### 20. Why is a primary key important during ingestion?

A primary key identifies a table row uniquely. It helps identify records, detect duplicates, connect related data and support safe incremental processing.

### 21. What is schema drift?

Schema drift is a change in the source structure over time, such as a new column, renamed field, changed data type or altered nested structure.

### 22. What would you do if a source team renamed a column?

Detect the change, inspect the impact, contact the source owner, update the source contract and transformation deliberately, test the change and deploy it through the approved process. Do not silently guess.

---

## 4. API source questions

### 23. What is an API?

An API is a defined interface that allows software systems to communicate. A Data Engineer may use an API to retrieve payment, shipment or customer data.

### 24. What is an endpoint?

An endpoint is a specific API address or resource through which a client makes a request.

### 25. What is an HTTP request?

An HTTP request is a message sent by a client to a server. It commonly contains a method, URL, headers and sometimes a body.

### 26. Explain GET and POST.

GET is commonly used to retrieve information. POST is commonly used to submit data or create a resource. Actual behavior is defined by the API contract.

### 27. What is an HTTP response?

A response is the server's reply. It commonly includes a status code, headers and a body containing data or error details.

### 28. What do HTTP 2xx, 4xx and 5xx mean?

2xx generally indicates success. 4xx indicates a request or client-side problem. 5xx indicates a server-side problem. The exact code and response body provide more detail.

### 29. What is API authentication?

Authentication verifies the identity of the caller. APIs may use API keys, bearer tokens, OAuth or certificates. Secrets must be stored securely.

### 30. What is pagination?

Pagination divides a large result into multiple pages. The client retrieves pages until the API indicates there is no more data.

### 31. What is rate limiting?

Rate limiting restricts how many requests a client can make in a given period. An API may return HTTP 429 when the limit is exceeded.

### 32. How would you handle a paginated API?

Read the documentation, authenticate securely, request the first page, validate its response, follow the next-page link or cursor, handle rate limits and retries, and persist progress so extraction can resume safely.

---

## 5. File format questions

### 33. What is CSV?

CSV stands for Comma-Separated Values. It is a text format commonly used to represent tabular records and columns.

### 34. What are the limitations of CSV?

CSV does not reliably preserve rich data types or nested structures. Delimiters, quoting, encoding, missing values and date formats must be agreed and validated.

### 35. What happens if a CSV value contains a comma?

The field must be quoted according to the CSV rules. Otherwise, the reader may interpret the comma as a column separator and shift the row structure.

### 36. What is JSON?

JSON stands for JavaScript Object Notation. It is a text format using objects, arrays and values. It is common in APIs and event messages.

### 37. What is the difference between a JSON object and an array?

An object is a collection of key-value pairs enclosed in curly braces. An array is an ordered list of values enclosed in square brackets.

### 38. Why is JSON useful for APIs?

JSON can represent nested objects, arrays and optional fields in a readable text format, making it convenient for software systems to exchange structured information.

### 39. What is XML?

XML stands for Extensible Markup Language. It represents hierarchical information using elements and attributes.

### 40. What is the difference between an XML element and an attribute?

An element is a named part of the document, commonly represented by opening and closing tags. An attribute provides additional information inside an element's opening tag.

### 41. What is a well-formed XML document?

A well-formed XML document follows XML syntax rules, including one root element, properly nested tags, matching opening and closing tags and correctly quoted attribute values.

### 42. What is an XML schema?

An XML schema, commonly described using XSD, defines allowed elements, attributes, structure and data types for an XML document.

### 43. What is Parquet?

Apache Parquet is a columnar binary file format commonly used for analytical storage and processing. It stores schema information and is designed to support efficient column-oriented access.

### 44. Why can Parquet be efficient for analytics?

Analytical queries often need only a subset of columns. Columnar storage can reduce unnecessary reads and can compress similar column values effectively. Actual performance depends on the query engine and data layout.

### 45. Why can't you normally read Parquet in a text editor?

Parquet is a binary format, not plain text. A compatible data processing or query tool is needed to inspect its records.

### 46. Compare CSV and Parquet.

CSV is human-readable text and convenient for simple tabular exchange. Parquet is a binary columnar format designed for analytical storage and efficient column reads.

### 47. Compare JSON and XML.

JSON represents objects and arrays using keys and values. XML represents hierarchy using tags and attributes. Both can represent nested data; the choice often depends on the source contract and integration requirements.

---

## 6. Logs and device source questions

### 48. What is an application log?

A log is a record of application or system activity, such as a request, warning, error or service event. Logs help investigate system behavior.

### 49. What is a structured log?

A structured log stores information in a consistent set of fields, often JSON. This makes filtering and aggregation easier.

### 50. What sensitive information should not be placed in logs?

Passwords, access tokens, secret keys, full payment credentials and unnecessary personal information should not be exposed. Follow data minimization and security policy.

### 51. What is device-generated data?

It is data automatically produced by devices or sensors, such as GPS locations, temperature readings or machine measurements.

### 52. What challenges occur with sensor data?

High volume, frequent timestamps, network outages, duplicate readings, late events, out-of-order events and inaccurate device clocks can affect processing.

---

## 7. Source inventory and governance questions

### 53. What is a Data Source Inventory?

It is a documented list of data sources and their important characteristics, such as owner, type, format, access method, schema, key, volume, refresh frequency, sensitivity and quality rules.

### 54. Why should every source have an owner?

The owner can confirm business meaning, access approval, expected changes, delivery schedules and support responsibilities. Without ownership, issues can remain unresolved.

### 55. What is a source contract?

A source contract is an agreement describing the structure, meaning and expected behavior of data exchanged between systems. It may define fields, types, keys, allowed values, delivery frequency and change rules.

### 56. What is data sensitivity?

Data sensitivity describes how carefully information must be protected. Customer details and financial information may require restricted access, encryption, masking and retention controls.

### 57. Why record refresh frequency?

It helps determine how often ingestion should run and whether the data can meet the business freshness requirement.

### 58. Why record volume?

Volume affects extraction strategy, network transfer, storage, partitioning, processing resources and cost.

### 59. What should you do when a source detail is unknown?

Record it as unknown or to be confirmed, identify the responsible source owner and obtain the approved answer. Do not invent values or assume a pattern.

---

## 8. Scenario-based interview questions

### 60. A supplier sends a CSV file with an unexpected delimiter. What do you do?

I would quarantine or hold the file, record the validation failure, compare it with the agreed file contract, contact the supplier owner and process it only after an approved correction or compatibility rule. Guessing the delimiter can shift columns and corrupt data.

### 61. A JSON API sometimes omits the phone field. What would you check?

I would check the API contract to see whether phone is optional, conditionally available or unexpectedly missing. I would validate required fields separately and represent optional missing values consistently.

### 62. An order JSON contains an array of 20 items. How would you make it usable in relational analytics?

I would preserve the parent order_id and create one item-level record per array element. I would validate item fields and maintain the relationship between the order and its items.

### 63. A file contains customer IDs such as 00045. Why must you be careful?

If converted to a number, the leading zeros may be removed. I would preserve it as text if the identifier's exact representation matters.

### 64. A Parquet file is uploaded but a text editor displays unreadable characters. Is the file corrupted?

Not necessarily. Parquet is a binary format. I would inspect it using a compatible tool and validate the schema and records.

### 65. The API returns HTTP 429. What does it mean?

The client has likely exceeded the provider's request rate limit. I would follow the API's retry guidance, respect any Retry-After information and avoid sending repeated requests too quickly.

### 66. A source team adds a new field. What should the pipeline do?

The pipeline should detect and assess the schema change. I would confirm the field's meaning and type with the source owner, update the contract and transformation, test compatibility and deploy through the approved process.

### 67. A delivery XML file contains a malformed closing tag. What would you do?

I would fail or quarantine that file according to the pipeline policy, record the parser error and source file details, contact the source owner and request a corrected file. I would not silently discard portions of the document.

### 68. A product image has no product ID in its metadata. What is the risk?

The image may not be reliably associated with a product. I would check the source contract and available manifest or catalog mapping before loading it as a product asset.

### 69. A database table contains 100 million records. Would you extract the entire table every hour?

Not automatically. I would inspect the business freshness requirement, update behavior, available change tracking, source impact and destination design. Incremental extraction may be more appropriate if reliable change tracking is available.

### 70. A source contains customer phone numbers, but the dashboard does not need them. What should you do?

Avoid copying or exposing unnecessary personal data. Apply data minimization and only include fields required by approved downstream use cases.

---

## 9. Mini-project interview: Data Source Inventory

### 71. Explain your Day 3 mini-project.

"I prepared an inventory for TechMart's customer database, order database, payment API, inventory CSV, delivery XML feed, website logs and product media storage. For each source, I documented the source type, format, owner, access method, key fields, refresh behavior, sensitivity, validation checks and failure contact. I marked unknown details for confirmation rather than assuming them."

### 72. Why did you include both business and technical owners?

The business owner explains the meaning and rules of the data. The technical owner supports access, system behavior and operational issues. Both perspectives are needed.

### 73. Why did you record primary or business keys?

Keys help identify records, connect related datasets, detect duplicates and design incremental ingestion.

### 74. Why did you document refresh frequency and latency?

They determine when data should be extracted and whether it can meet the consumer's freshness requirement.

### 75. What would you do if a source has no documented schema?

I would inspect representative data only through approved access, contact the owner, document observed fields and confirm their meanings and types. I would not treat a sample as a complete contract.

### 76. What is the most important outcome of the inventory?

It gives the engineering team a clear view of source ownership, structure, access, update behavior, quality and risk before implementation begins.

---

## 10. Difference-based quick revision

| Compare | Key difference |
|---|---|
| Source vs destination | Where data originates vs where it is delivered |
| Field vs record | One named value vs collection of related values |
| Structured vs semi-structured | Fixed tabular structure vs keys/tags and flexible nesting |
| Semi-structured vs unstructured | Explicit structural markers vs no consistent tabular organization |
| CSV vs JSON | Delimited rows and columns vs objects and arrays |
| JSON vs XML | Key-value objects/arrays vs tags and attributes |
| CSV vs Parquet | Human-readable text rows vs binary columnar storage |
| Element vs attribute | XML content node vs metadata inside an opening tag |
| Full vs incremental extraction | Read all selected records vs only new/changed records |
| Event time vs processing time | When it happened vs when it was handled |

---

## 11. Common weak answers and stronger answers

| Weak answer | Stronger answer |
|---|---|
| "A source is a database." | "A source can be a database, API, file, log, device or another system." |
| "JSON is just a text file." | "JSON is a text data format with objects, arrays and typed values." |
| "CSV always stores numbers as numbers." | "CSV is text; types must be defined or inferred carefully." |
| "Parquet is a compressed CSV." | "Parquet is a schema-aware columnar binary format." |
| "Every JSON field is mandatory." | "The source contract defines required and optional fields." |
| "We can guess a new column's meaning." | "Confirm the field definition and update the contract deliberately." |
| "The file extension tells us the format." | "Validate the actual file content and structure." |
| "Unknown source details can be assumed." | "Mark them for confirmation with the source owner." |

---

## 12. Practice tasks

### Task 1 — Classify the data

Classify each as structured, semi-structured or unstructured:

1. PostgreSQL customer table.
2. Nested payment JSON.
3. XML shipment document.
4. Product photograph.
5. Audio customer support recording.
6. CSV inventory export.
7. JSON application event.

Explain your reasoning.

### Task 2 — Compare formats

Explain when you would use CSV, JSON, XML and Parquet. Include one limitation of each.

### Task 3 — Inspect a source

Choose one source and list the owner, access method, schema, key, volume, refresh frequency, sensitivity and quality rules you need to confirm.

### Task 4 — Mini-project defense

Present the Data Source Inventory and explain how it reduces risk before pipeline development.

---

## 13. Final interview checklist

- [ ] I can explain source, destination, field, record and schema.
- [ ] I can distinguish structured, semi-structured and unstructured data.
- [ ] I can describe databases, APIs, files, logs and devices as sources.
- [ ] I understand CSV quoting, delimiters and missing values.
- [ ] I can explain JSON objects, arrays and nested records.
- [ ] I can explain XML elements, attributes and well-formedness.
- [ ] I can explain Parquet and columnar storage.
- [ ] I know why identifiers should not always be numeric.
- [ ] I understand schema drift and source contracts.
- [ ] I can prepare and defend a Data Source Inventory.

**Interview rule:** A good Data Engineer does not only ask "Can I read this file?" They ask "Who owns it, what does it mean, how does it change, how do I validate it, and am I allowed to use it?"
