# Day 5: Data Pipeline Fundamentals

**Phase:** 1 — Data Engineering Foundations  
**Learning time:** 2–3 hours  
**Mini-project:** Pipeline Requirements Document

## 1. Learning outcomes

- Explain a pipeline lifecycle from requirements to operations.
- Understand reliability, scalability and fault tolerance.
- Recognize common pipeline failure modes.
- Understand data governance, security and ownership.
- Write practical requirements and acceptance criteria for a pipeline.

## 2. What is a data pipeline?

A data pipeline is a repeatable process that moves data from one or more sources through defined processing steps into a destination.

Example:

`Read new orders -> Validate -> Standardize -> Calculate sales -> Load warehouse -> Reconcile -> Report status`

A production pipeline is more than code that runs once. It must handle new data, failures, retries, changing schemas and operational support.

## 3. Pipeline lifecycle

### Stage 1 — Discover requirements

Understand:

- Business purpose.
- Source and target systems.
- Data owner and consumer.
- Required fields and definitions.
- Expected volume and growth.
- Refresh frequency and latency.
- Security classification.
- Retention requirements.
- Acceptance criteria.

Do not begin by selecting a tool. First understand what the business needs.

### Stage 2 — Profile the source

Inspect sample data and source behavior:

- Schema and data types.
- Primary or natural keys.
- Null and duplicate patterns.
- Update and delete behavior.
- Historical availability.
- Rate limits or extraction constraints.
- Expected arrival time.

### Stage 3 — Design the pipeline

Decide:

- Full or incremental extraction.
- Batch or streaming.
- Raw retention.
- Validation rules.
- Transformation location.
- Target model.
- Idempotency strategy.
- Retry and recovery behavior.
- Monitoring and alerting.

### Stage 4 — Build and test

Implement small, testable steps. Test normal records, invalid records, empty input, duplicates, late data, schema changes and destination failures.

### Stage 5 — Deploy and operate

Use version control and a controlled deployment process. Monitor actual runs, compare outputs with expectations and maintain a recovery runbook.

## 4. Reliability

**Reliability** means the pipeline produces correct and complete results consistently and behaves predictably when something goes wrong.

Reliability includes:

- Correct extraction.
- Duplicate-safe loading.
- Retry behavior.
- Checkpointing.
- Recovery.
- Data validation.
- Monitoring.
- Clear ownership.

A job that completes successfully but silently omits half the source data is not reliable.

## 5. Common failure categories

| Category | Example | Detection |
|---|---|---|
| Source failure | Database unavailable | Connection error and source health |
| Network failure | API timeout | Request status and retry metrics |
| Input failure | Missing daily file | Arrival check |
| Data failure | Invalid date value | Schema/type validation |
| Transformation failure | Unexpected null in logic | Tests and exception details |
| Target failure | Warehouse unavailable | Load status |
| Capacity failure | Memory exhausted | Resource metrics |
| Scheduling failure | Dependency not completed | Orchestrator state |

Each failure should have a defined response, owner and recovery path.

## 6. Retries and backoff

A retry attempts an operation again after a failure.

Retries are useful for temporary issues such as network timeouts or service throttling. They do not fix permanent errors such as an invalid schema or missing required column.

Use bounded retries and increasing delays (backoff). Avoid retrying forever.

Before retrying a write operation, understand whether it is safe to repeat. A retry must not duplicate business effects.

## 7. Idempotency

An operation is **idempotent** when repeating it with the same input produces the same final state as running it once.

Example:

- Unsafe: append the same order rows every time a job runs.
- Safer: merge records using a stable order key and update only the intended target state.

Idempotency is essential for safe retries, backfills and duplicate file delivery.

## 8. Checkpointing and recovery

A checkpoint records progress so processing can resume from a known safe point.

Examples:

- Last successfully processed source timestamp.
- API page token.
- Maximum committed source ID.
- Kafka consumer offset.
- Completed file or partition list.

A checkpoint must advance only after the corresponding work is safely committed. Otherwise, data can be skipped or duplicated.

A recovery plan should specify:

1. What failed.
2. Which data was committed.
3. Where processing should resume.
4. Whether replay is safe.
5. How the output will be reconciled.

## 9. Fault tolerance

Fault tolerance is the ability of a system to continue operating or recover when components fail.

Common techniques:

- Replication.
- Durable storage.
- Retries.
- Checkpoints.
- Dead-letter or quarantine paths.
- Redundant services.
- Backups.
- Recovery testing.

Not every junior pipeline needs distributed high availability. The design should match business impact and recovery objectives.

## 10. Scalability

Scalability is the ability to handle increasing workload.

Workload can grow through:

- More rows.
- Larger files.
- More sources.
- More frequent runs.
- More concurrent consumers.
- More complex transformations.

Ways to improve scalability:

- Incremental extraction instead of repeated full scans.
- Partition large datasets.
- Process independent partitions in parallel.
- Use efficient columnar formats for analytics.
- Avoid unnecessary data copies.
- Push suitable transformations to the database or warehouse.
- Monitor resource use and bottlenecks.

Parallelism can increase cost and complexity. It is not automatically faster for every workload.

## 11. Performance and cost basics

Measure before optimizing.

Useful measures:

- Runtime.
- Records per second.
- Bytes read and written.
- CPU and memory.
- API request count.
- Warehouse query duration.
- Storage growth.
- Cost per run or per data volume.

A faster pipeline may be more expensive. A cost-efficient pipeline that misses the freshness target is also not acceptable. Balance both against business requirements.

## 12. Data governance

Data governance defines how data is owned, understood, protected, accessed and retained.

Key elements:

- Data ownership and stewardship.
- Business definitions.
- Metadata and catalog.
- Data classification.
- Access control.
- Quality rules.
- Lineage.
- Retention and deletion.
- Auditability.

Example: customer email addresses may be classified as personal data. Access should be limited to approved roles and downstream exports should follow company policy.

## 13. Security fundamentals

### Least privilege
Give a user or service only the permissions needed for its task.

### Secret management
Do not hardcode passwords, API keys or tokens in source code or notebooks. Use approved secret storage or secure runtime configuration.

### Encryption
Protect data in transit and at rest using approved platform capabilities.

### Network controls
Restrict which services can communicate with databases and storage.

### Data minimization
Collect and expose only fields needed for the business purpose.

### Audit
Record sensitive access and important administrative actions according to policy.

## 14. Pipeline observability

A pipeline should emit enough evidence to answer:

- Did the run start?
- Did it finish?
- How long did it take?
- How many records were read, accepted and rejected?
- Which source and target were used?
- What was the data freshness?
- What failed?
- Who was alerted?

Useful run metadata:

| Field | Example |
|---|---|
| run_id | Unique execution ID |
| pipeline_name | daily_orders |
| source | orders database |
| start_time | UTC timestamp |
| end_time | UTC timestamp |
| status | SUCCESS / FAILED |
| rows_read | 10,000 |
| rows_written | 9,998 |
| rows_rejected | 2 |
| error_summary | Invalid order date |

Avoid logging passwords, access tokens or unnecessary personal data.

## 15. Define service expectations

A requirement should be measurable.

Weak: “The pipeline should be fast.”

Better: “The daily sales dataset must be available by 7:00 AM local business time on at least 99% of scheduled business days.”

Useful service expectations:

- Freshness target.
- Availability or success rate.
- Maximum runtime.
- Recovery time objective (RTO).
- Recovery point objective (RPO).
- Retention period.
- Data quality thresholds.

RTO is how quickly service should be restored. RPO describes the acceptable amount of data loss measured in time or recovery point.

## 16. Mini-project: Pipeline Requirements Document

### Scenario

A retailer wants a daily order pipeline.

Business requirements:

- Load paid and cancelled orders for analysis.
- Dashboard must be ready by 7 AM.
- Source is PostgreSQL.
- Target is a warehouse.
- Updates to existing orders must be reflected.
- Duplicate processing must not inflate revenue.
- Failures must alert the data team.

### Step 1 — Business purpose

Create a trusted daily order dataset for finance and operations reporting.

### Step 2 — Source and target

| Item | Definition |
|---|---|
| Source | PostgreSQL orders and order_items |
| Source owner | Retail application team |
| Target | Warehouse sales order model |
| Consumer | Finance and operations BI |
| Business key | order_id + order_item_id for line-level records |

Confirm actual keys and table relationships with the source owner before implementation.

### Step 3 — Data requirements

Required fields:

- order_id
- order_item_id
- customer_id
- product_id
- order_timestamp
- quantity
- unit_price
- order_status
- updated_at

Rules:

- IDs must not be null.
- Quantity must be positive.
- Unit price must be non-negative.
- Timestamp must be valid.
- Status must be from the approved list.
- Duplicate line keys must be detected.
- Updated records must replace or version the prior target state according to the model.

### Step 4 — Operational requirements

| Requirement | Acceptance criterion |
|---|---|
| Freshness | Target ready by 7 AM |
| Duplicate safety | Reprocessing a batch does not increase totals |
| Completeness | Source/target counts reconciled with documented exclusions |
| Recovery | Failed run can resume or rerun safely |
| Monitoring | Run status and row metrics recorded |
| Alerting | Team notified after retry policy is exhausted |
| Security | Credentials are not stored in code |

### Step 5 — Failure matrix

| Failure | Detection | Response |
|---|---|---|
| Source unavailable | Connection check | Bounded retry and alert |
| Invalid record | Validation | Quarantine with reason |
| Duplicate delivery | Batch/key check | Idempotent merge |
| Target unavailable | Load failure | Retry safely |
| Schema changed | Contract validation | Stop and notify owner |

### Step 6 — Architecture flow

`PostgreSQL -> Incremental extraction -> Raw landing -> Validation -> Transformation -> Warehouse merge -> Reconciliation -> BI`

Monitoring and alerting observe every stage.

### Step 7 — Deliverable

Write a document with:

1. Business objective.
2. Source and target inventory.
3. Field definitions.
4. Ingestion strategy.
5. Transformation rules.
6. Quality checks.
7. Security requirements.
8. Reliability and recovery design.
9. Monitoring and alerting.
10. Acceptance criteria and open questions.

## 17. Common mistakes

| Mistake | Better approach |
|---|---|
| Retrying permanent errors | Classify transient and permanent failures |
| Updating checkpoint before commit | Advance only after safe completion |
| Appending every retry | Use idempotent writes |
| Logging credentials | Mask secrets and sensitive fields |
| Claiming reliability from job status | Reconcile data and monitor quality |
| Optimizing without measurements | Profile first |
| No owner for alerts | Define operational ownership |
| No recovery test | Exercise the runbook |

## 18. Day 5 revision

- A pipeline has a lifecycle from requirements through operations.
- Reliability includes correctness, recovery and monitoring.
- Retries need bounds and safe write behavior.
- Idempotency prevents duplicate business effects.
- Checkpoints support safe resumption.
- Scalability must balance performance and cost.
- Governance defines ownership, quality, access and retention.
- Security must be built into the design.

## 19. Check your understanding

1. Why is a successful task status not enough to prove data correctness?
2. What makes a pipeline operation idempotent?
3. When should a checkpoint advance?
4. Why should permanent errors not be retried indefinitely?
5. What is the difference between RTO and RPO?
6. Which metrics would you include in a pipeline run record?
