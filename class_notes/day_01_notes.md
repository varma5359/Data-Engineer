# Day 1 — Introduction to Data Engineering
**Pathway:** TechPhile.ai Data Engineer | **Phase:** Foundations | **Level:** Beginner

## Learning objectives
By the end, you can explain what a Data Engineer does, distinguish Data Engineer/Analyst/Scientist responsibilities, trace data through a typical architecture, and describe the data lifecycle.

## 1. What is data?
Data is a recorded observation about something: an order, customer, payment, click, sensor reading, or support ticket.

Example: an online order may produce:
- order_id: 501
- customer_id: 1008
- order_date: 2026-11-10
- amount: 2499.00
- status: delivered

A business application creates data while people and systems perform work.

## 2. What is Data Engineering?
**Data Engineering is the discipline of collecting data from source systems, moving it safely, transforming it into trustworthy and usable datasets, and maintaining the systems that deliver those datasets.**

A Data Engineer builds the data foundation that analytics, reporting, and machine-learning teams depend on.

### Why is this role needed?
Imagine a retailer has customer data in PostgreSQL, inventory in CSV files, and sales in an API. Each source has a different structure and update time. Analysts cannot reliably combine them manually every morning.

A Data Engineer creates a repeatable pipeline:
`Sources → Ingestion → Storage → Transformation → Data Serving`

The result is consistent, documented data that can be queried by business teams.

## 3. Data Engineer responsibilities
| Responsibility | Meaning | Example |
|---|---|---|
| Ingestion | Bring data from sources | Read sales API every hour |
| Transformation | Clean and reshape data | Standardize dates and currency |
| Storage | Choose and manage destinations | Load curated tables into a warehouse |
| Modeling | Organize data for analysis | Create sales fact and customer dimension |
| Quality | Detect incorrect/incomplete data | Reject rows with missing order IDs |
| Reliability | Keep pipelines dependable | Retry temporary API failures |
| Performance | Process data efficiently | Partition a large transaction table |
| Security | Protect data and access | Use least-privilege database credentials |
| Operations | Observe and maintain pipelines | Monitor failures and processing counts |

**A Data Engineer is not only a person who writes ETL scripts.** The job includes design, testing, security, monitoring, recovery, and communication.

## 4. Data Engineer vs Data Analyst vs Data Scientist
| Role | Main question | Typical work | Common output |
|---|---|---|---|
| Data Engineer | Can reliable data be delivered? | Ingestion, pipelines, storage, quality | Tables, datasets, pipelines |
| Data Analyst | What happened and why? | SQL analysis, dashboards, business metrics | Reports, dashboards, insights |
| Data Scientist | What may happen / what pattern exists? | Statistics, experimentation, predictive models | Forecasts, models, experiments |

These roles collaborate. A Data Engineer may build a trusted sales table; an Analyst uses it to explain revenue; a Scientist uses it to predict demand.

## 5. Typical industry data architecture
`Operational Sources → Ingestion → Raw Storage → Transformation → Curated/Warehouse → BI / ML / Applications`

### Layer 1 — Sources
Examples: application databases, REST APIs, CSV/Excel files, logs, CRM, ERP, IoT devices.

### Layer 2 — Ingestion
Extract or receive data. Common patterns:
- Batch: collect records at intervals.
- Streaming: process events continuously or in small time windows.
- Full load: read the complete dataset.
- Incremental load: read only new or changed records.

### Layer 3 — Raw storage
Keep source data close to its original form. Raw data helps replay processing, investigate incidents, and compare source vs destination.

### Layer 4 — Transformation
Apply business rules, type conversion, deduplication, validation, joins, and aggregation.

### Layer 5 — Curated / warehouse
Publish clean, organized tables with clear definitions and ownership.

### Layer 6 — Consumers
BI dashboards, analysts, data scientists, applications, and operational reports consume the published data.

## 6. Data lifecycle
`Create → Collect → Store → Validate → Transform → Publish → Consume → Monitor → Retain/Archive/Delete`

- **Create:** a source application records an order.
- **Collect:** pipeline extracts or receives the record.
- **Store:** raw copy is retained.
- **Validate:** required fields and types are checked.
- **Transform:** business logic is applied.
- **Publish:** approved table becomes available.
- **Consume:** dashboard calculates revenue.
- **Monitor:** pipeline freshness and failures are checked.
- **Retain/delete:** data follows retention and privacy policy.

## 7. Real-time scenario — daily sales dashboard
Business request: “Show yesterday's revenue by region at 9 AM.”

Engineering questions:
1. Which system is the source of truth for orders?
2. Are cancelled/refunded orders included?
3. Which timezone defines yesterday?
4. How are late-arriving orders handled?
5. What happens if the source API is unavailable?
6. How do we detect duplicate orders?
7. Who can access customer-level information?
8. How will the dashboard know the dataset is fresh?

This is why requirements, quality, reliability, and business definitions matter—not only code.

## 8. Mini-project — Data Engineering Architecture Map
**Deliverable:** a one-page diagram for a retail company.

Include:
- Sources: customer database, product CSV, sales API.
- Ingestion: database extraction, file ingestion, API extraction.
- Raw layer: original records with ingestion timestamp.
- Transformation: standardize IDs/dates, remove duplicates, validate.
- Warehouse: customers, products, sales tables.
- Consumers: sales dashboard and analyst SQL queries.
- Controls: access permissions, logs, quality checks, alerts.

Use arrows to show data movement. Label batch or streaming where relevant.

## 9. Common beginner mistakes
- Thinking Data Engineering means only SQL or only Python.
- Treating a successful script run as proof that data is correct.
- Ignoring duplicates, late data, missing values, and schema changes.
- Loading raw sensitive data into broadly accessible locations.
- Building a pipeline without monitoring or recovery.
- Confusing a dashboard/report with the pipeline that supplies it.

## Quick revision
- Data Engineer: builds and operates reliable data systems.
- Pipeline: repeatable movement and processing of data.
- Raw layer: source-preserving data; curated layer: trusted consumption-ready data.
- Reliability, quality, security, and observability are part of engineering.
- Always connect technical design to a business requirement.
