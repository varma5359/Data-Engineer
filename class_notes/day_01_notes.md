# Day 1: Introduction to Data Engineering

**Phase:** 1 — Data Engineering Foundations  
**Learning time:** 2–3 hours  
**Mini-project:** Data Engineering Architecture Map

## 1. Learning outcomes

By the end of this lesson, you should be able to:

- Explain what data engineering means in a business.
- Describe the responsibilities of a junior Data Engineer.
- Distinguish a Data Engineer from a Data Analyst and Data Scientist.
- Trace data from its original source to a trusted analytical destination.
- Recognize the main components in a modern data platform.
- Explain why reliability, data quality, security and documentation matter.

## 2. Start with a business problem

Imagine an online retail company. Customers place orders through a website and mobile application. Payments are processed by a payment provider. Warehouse staff update inventory. Marketing stores campaign information in another system.

The management team asks:

- How much revenue did we earn yesterday?
- Which products are running out of stock?
- Which customers have not purchased in 90 days?
- Which marketing campaign generated the most orders?

The information exists, but it is spread across different systems. The company needs a dependable way to collect, combine, clean, store and deliver it.

That engineering work is the foundation of data engineering.

## 3. What is data engineering?

**Data engineering** is the discipline of designing, building, testing and operating systems that move data from source systems into reliable, usable destinations.

A Data Engineer makes data:

1. Available — collect it from the systems where it is created.
2. Understandable — apply consistent names, formats and business rules.
3. Trustworthy — validate it and detect missing or incorrect records.
4. Accessible — store it where analysts and applications can use it.
5. Reliable — recover from failures and prevent duplicate or lost data.
6. Governed — protect sensitive information and document its meaning.

A Data Engineer does not simply copy files. The engineer creates a repeatable system that can run safely as data volume and business needs grow.

## 4. Why companies need Data Engineers

| Business situation | Engineering responsibility |
|---|---|
| Orders are in a transactional database | Extract order records safely |
| Payments arrive from an external API | Connect to the API and handle failures |
| Files arrive every night | Detect, validate and ingest new files |
| Different systems use different customer IDs | Resolve relationships and model the data |
| Reports show different revenue totals | Reconcile sources and standardize business rules |
| A pipeline fails at 2 AM | Alert, investigate and recover without corrupting data |
| Customer information is sensitive | Apply access control and data protection |

**Business outcome:** teams spend less time correcting spreadsheets and more time making decisions from consistent data.

## 5. Data Engineer responsibilities

### 5.1 Understand data sources

A source is the system or location where data originates.

Examples:

- MySQL or PostgreSQL databases.
- REST APIs from payment, CRM or shipping providers.
- CSV and Excel files from business teams.
- JSON event messages from applications.
- Application logs and clickstream events.
- Cloud object storage.

Before connecting to a source, ask:

- Who owns the data?
- What does each field mean?
- How frequently does it change?
- Can records be updated or deleted?
- What is the expected volume?
- Does it contain personal or confidential information?

### 5.2 Ingest data

**Ingestion** means bringing data from a source into a platform.

Common patterns:

- Full load: read the complete dataset.
- Incremental load: read only new or changed records.
- Batch: process a group of records on a schedule.
- Streaming: process events continuously or in small time windows.

A junior engineer should know why a pattern was selected, not only how to write the extraction code.

### 5.3 Transform data

Raw data is often unsuitable for direct reporting.

Example source records:

| order_id | quantity | unit_price | status |
|---|---:|---:|---|
| 101 | 2 | 500 | paid |
| 102 | 1 | 800 | cancelled |
| 103 | 3 | 200 | paid |

A business rule may define recognized sales as paid orders only.

The transformation process might:

- Convert text dates into a standard date type.
- Remove exact duplicate deliveries.
- Reject records with invalid order identifiers.
- Calculate line revenue.
- Standardize status values.
- Keep cancelled orders available for operational analysis while excluding them from recognized sales.

Do not silently delete records simply because they are inconvenient. Preserve raw data and document the rule.

### 5.4 Store and publish data

Data may be stored in:

- A landing area for received data.
- A raw layer that preserves source records.
- A cleaned layer with standardized fields.
- A warehouse or data mart for analytical queries.

The exact architecture varies by company. The important idea is to make each layer's purpose clear.

### 5.5 Operate pipelines

A pipeline is a repeatable sequence of data-processing steps.

Operating it includes:

- Scheduling and dependency management.
- Monitoring success, failure and duration.
- Logging useful execution details.
- Retrying temporary failures safely.
- Tracking processed records.
- Alerting the responsible team.
- Documenting recovery procedures.

## 6. Data Engineer vs Data Analyst vs Data Scientist

| Area | Data Engineer | Data Analyst | Data Scientist |
|---|---|---|---|
| Main focus | Data systems and pipelines | Business questions and insights | Statistical and predictive models |
| Typical input | Databases, APIs, files, events | Curated datasets and warehouse tables | Prepared datasets and features |
| Typical output | Reliable datasets and platforms | Reports, dashboards and analysis | Experiments, forecasts and models |
| Main questions | Can data arrive correctly and on time? | What happened and why? | What may happen and what action is useful? |
| Common tools | Python, SQL, Spark, Airflow, cloud | SQL, Excel, Power BI, Python | Python, statistics, ML frameworks |

These roles collaborate. The boundaries can overlap depending on the organization.

### Example: late deliveries

- Data Engineer builds a pipeline that combines orders, warehouse scans and courier events.
- Data Analyst measures late-delivery rates by region and carrier.
- Data Scientist may predict which deliveries are at risk of being late.

## 7. A simple modern data architecture

`Source systems -> Ingestion -> Raw storage -> Transformation -> Warehouse -> Consumers`

### Component responsibilities

| Component | Purpose | Example |
|---|---|---|
| Source | Creates operational data | Order database |
| Ingestion | Moves source data | Python API extractor |
| Landing/raw | Preserves received records | S3 raw bucket |
| Transformation | Applies quality and business rules | SQL or Spark |
| Warehouse | Organizes analytical data | PostgreSQL warehouse |
| Orchestration | Runs steps in order | Airflow |
| Observability | Shows health and failures | Logs, metrics, alerts |
| Consumer | Uses trusted data | BI dashboard |

A real architecture can contain several sources, queues, databases and processing engines. This diagram is a learning model, not a mandatory product stack.

## 8. Follow one order through the platform

Suppose order 501 is placed at 10:05 AM.

1. The website writes the order to the operational database.
2. An ingestion job reads the new order.
3. The raw layer stores the received record with source and ingestion metadata.
4. Validation checks the order ID, timestamp, quantity and amount.
5. Transformation standardizes status and calculates line amount.
6. The warehouse updates its order fact table.
7. A dashboard refreshes and includes the order in today's sales.

At each step, ask what happens if the process fails. For example, if the warehouse load fails after raw storage succeeds, the pipeline should be able to resume without duplicating the order.

## 9. Important engineering qualities

### Reliability
The pipeline produces the expected result consistently and handles failures predictably.

### Scalability
The design can handle growth in records, users and processing demand without requiring a complete redesign.

### Maintainability
Another engineer can understand, troubleshoot and safely change the code.

### Data quality
The data meets defined expectations for completeness, validity, consistency, uniqueness and timeliness.

### Security
Only authorized people and services can access the data they need.

### Observability
The team can understand what the pipeline did, how long it took and where it failed.

### Cost awareness
Storage, compute, network transfer and repeated processing all have costs. A good design avoids waste without sacrificing correctness.

## 10. What a junior Data Engineer does in practice

A junior engineer may be asked to:

- Add a new column to an ingestion process.
- Load a daily CSV file into a database.
- Fix a date parsing error.
- Add a row-count validation.
- Investigate why yesterday's load is missing.
- Write a SQL query to compare source and target totals.
- Update pipeline documentation.
- Add a unit test for a transformation.
- Follow a runbook to rerun a failed job.

The expected behavior is to understand the impact, test the change, communicate risks and avoid changing production data casually.

## 11. Mini-project: Data Engineering Architecture Map

### Scenario

A retail company uses:

- PostgreSQL for orders and customers.
- A payment API for payment status.
- CSV files for daily inventory.
- JSON files for website events.
- A warehouse for reporting.
- A BI dashboard for management.

### Step 1 — Identify the sources

| Source | Data | Format / access | Refresh |
|---|---|---|---|
| PostgreSQL | Orders and customers | Relational tables | Every 15 minutes |
| Payment API | Payment status | REST / JSON | Every 10 minutes |
| Inventory team | Stock snapshot | CSV file | Daily |
| Website | Click events | JSON events | Near real time |

Refresh frequencies are scenario assumptions. A production design must confirm them with source owners.

### Step 2 — Draw the flow

`PostgreSQL ----\`
  
`Payment API -----+-> Ingestion -> Raw zone -> Validation and transformation -> Warehouse -> BI`

`Inventory CSV --/`

`Website events --------------------------/`

### Step 3 — Define responsibilities

| Stage | Responsibility | Evidence to retain |
|---|---|---|
| Extract | Read data safely | Source name, extraction time |
| Land | Preserve received data | File or batch identifier |
| Validate | Check required fields and types | Validation results |
| Transform | Apply documented rules | Transformation version |
| Load | Write to target | Rows inserted/updated |
| Monitor | Detect failures and delays | Run status and metrics |

### Step 4 — Add failure questions

- What if the API returns HTTP 429?
- What if the CSV is missing?
- What if the same file arrives twice?
- What if an order is updated after the first load?
- What if the warehouse is unavailable?
- Who receives an alert?
- How can the team safely rerun the job?

### Step 5 — Deliverable

Create an architecture diagram containing:

- Four source systems.
- Ingestion method for each source.
- Raw storage.
- Validation and transformation.
- Warehouse.
- BI consumer.
- Monitoring and security notes.

A good diagram labels the purpose of each component and the direction of data flow.

## 12. Common mistakes

| Mistake | Why it is harmful | Better practice |
|---|---|---|
| Thinking data engineering is only SQL | Ignores ingestion, operations and reliability | Learn the full data lifecycle |
| Copying source data without validation | Errors spread downstream | Define checks at ingestion boundaries |
| Overwriting raw data | Removes evidence needed for debugging | Preserve immutable or versioned raw inputs |
| Assuming a job ran because it was scheduled | Schedules do not guarantee success | Monitor actual run outcomes |
| Ignoring duplicate delivery | Can inflate business totals | Design duplicate-safe processing |
| Storing passwords in code | Exposes credentials | Use a secret manager or secure environment |
| Building without documentation | Makes support difficult | Record ownership, schema and recovery steps |

## 13. Day 1 revision

- Data engineering builds and operates systems that deliver reliable data.
- A pipeline moves and transforms data through defined stages.
- Sources can be databases, APIs, files and events.
- Data Engineers enable analysts, scientists and applications.
- Reliability, quality, security, observability and cost matter from the beginning.
- A good architecture explains both the normal flow and failure recovery.

## 14. Check your understanding

1. Why does a company need a Data Engineer if it already has a database?
2. What is the difference between a source and a warehouse?
3. Why should raw data be preserved?
4. What could happen if a pipeline processes the same order twice?
5. Which team should define the meaning of a business field?
6. What evidence would help investigate a failed daily load?

**Strong answer principle:** explain the business problem, the engineering action and how you would verify the result.
