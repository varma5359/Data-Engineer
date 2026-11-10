# Day 5 Interview Preparation — Data Pipeline Fundamentals

**Phase:** 1 — Data Engineering Foundations  
**Mini-project:** Pipeline Requirements Document  
**Audience:** Freshers and junior Data Engineer candidates

---

# How to use this interview guide

For each question:

1. Read the question.
2. Try answering in your own words.
3. Give a small real-world example.
4. Explain why the concept matters to a Data Engineer.
5. Practise the follow-up question.

Do not memorize long answers word for word. Understand the concept and explain it clearly.

# Section 1 — Core fundamentals

## 1. What is a data pipeline?

A data pipeline is a repeatable sequence of steps that collects data from sources, validates and transforms it, and delivers it to a destination.

**Example:** Read orders from a database, payment details from an API, combine them and load a daily sales table.

**Follow-up:** Is every file copy a data pipeline?

A file copy can be one pipeline step, but a dependable production pipeline normally also needs defined processing, validation, delivery and operational behavior.

## 2. Why do organizations build data pipelines?

To automate repeatable data movement and processing, reduce manual work, improve consistency and make data available to reporting, analytics or applications.

## 3. What are the main components of a pipeline?

Sources, extraction, validation, transformation, loading, destination, scheduling or triggering, monitoring and operational support.

## 4. What is a pipeline lifecycle?

It is the process of understanding requirements, profiling sources, designing, building, testing, deploying, monitoring and improving a pipeline.

## 5. Why should a Data Engineer understand the business requirement first?

Because technical choices depend on what the business needs. The engineer must understand definitions, freshness, expected volume, quality rules, security and acceptable failure behavior before choosing tools.

## 6. What is source profiling?

Examining a source's schema, data types, volume, nulls, duplicates, keys, update behavior, history and availability before building an ingestion process.

## 7. What is the difference between a business requirement and a technical requirement?

A business requirement describes the outcome the business needs. A technical requirement describes how the system must behave to deliver that outcome.

**Example:** "Sales report by 7 AM" is a business need. "The pipeline must complete the previous business date's load before 7 AM" is a technical requirement.

## 8. Why are data definitions important?

Terms such as revenue, active customer and completed order can have different meanings across teams. The Data Engineer must document the approved definition rather than inventing one.

# Section 2 — Extraction and processing

## 9. What is full extraction?

Reading the complete dataset from a source.

**Use case:** A small reference table that is refreshed in full.

**Limitation:** It can repeatedly process large amounts of unchanged data.

## 10. What is incremental extraction?

Reading only records that are new or changed since a known point.

**Example:** Read orders whose updated_at value is later than the last successful watermark.

## 11. What is a watermark?

A saved progress value used to identify the point from which incremental processing should continue. It may be a timestamp, source ID or another cursor.

**Follow-up:** Can a timestamp watermark miss records?

Yes. Unreliable timestamps, late-arriving records and equal timestamps can create gaps. Designs may use overlap windows, composite cursors, change tracking and reconciliation.

## 12. What is batch processing?

Processing a group of records together at scheduled or defined intervals.

**Example:** Process yesterday's orders at 2 AM.

## 13. What is streaming?

Processing events continuously or as they arrive, often with low-latency requirements.

**Example:** Update an order dashboard when a new order event is received.

## 14. Batch vs streaming — what is the difference?

Batch processes accumulated groups of records. Streaming processes ongoing event flows. Batch is often simpler for periodic reports; streaming is useful when the business needs fresher information.

## 15. What is ETL?

Extract, Transform, Load. Data is transformed before being loaded into the target.

## 16. What is ELT?

Extract, Load, Transform. Data is loaded into the target platform first and transformed there.

## 17. How do you choose between batch and streaming?

Understand required freshness, data volume, event frequency, operational complexity, cost and consumer needs. Do not choose streaming simply because it is a popular technology.

# Section 3 — Reliability and failure handling

## 18. What does pipeline reliability mean?

The pipeline consistently delivers correct and complete results and behaves predictably when problems occur.

## 19. Is a successful job status proof of correct data?

No. The job may finish but still miss records, duplicate data or apply incorrect business rules. Check quality, counts, totals, freshness and reconciliation.

## 20. What is idempotency?

Repeating the same logical operation does not create additional unintended effects.

**Example:** Retrying an order batch should not insert the same order twice.

## 21. Why is idempotency important?

Pipelines fail and retry. Idempotency helps make replay safe and prevents duplicate business effects.

## 22. How can you make a load idempotent?

Use appropriate stable keys, unique constraints, merge/upsert logic, batch identifiers, deduplication rules and safe transaction boundaries.

**Follow-up:** Can customer_id be the unique key for an order table?

Usually not. One customer can place multiple orders. Use the key that identifies the order or business event.

## 23. What is checkpointing?

Saving processing progress so a pipeline can resume from a known safe point after interruption.

## 24. Why should a checkpoint not advance before a destination write is committed?

If the checkpoint advances and the write fails, the next run may skip records that were never safely loaded.

## 25. What is a retry?

Attempting an operation again after a failure.

## 26. Should every error be retried?

No. Temporary network or service errors may be retried. Invalid SQL, invalid credentials and malformed data generally require correction rather than repeated attempts.

## 27. What is exponential backoff?

A retry strategy that increases the wait between attempts, for example 1, 2, 4 and 8 seconds. Production systems commonly use limits and jitter too.

## 28. What is fault tolerance?

The ability of a system to continue operating or recover when a component fails.

## 29. Reliability vs availability vs fault tolerance?

- Reliability: consistent correct results.
- Availability: service is accessible when needed.
- Fault tolerance: system can continue or recover despite component failures.

## 30. What is a dead-letter queue?

A location where messages that cannot be processed after defined attempts are placed for investigation or later handling.

## 31. What is a quarantine table?

A controlled location for invalid or rejected records, often including the reason for rejection.

## 32. What is recovery?

Restoring normal operation and correct data after a failure. Recovery may involve fixing the issue, safely replaying data, validating results and communicating status.

## 33. What is a backfill?

A controlled processing run for historical data that was missed or needs rebuilding.

**Follow-up:** Is backfill the same as retry?

No. A retry repeats a failed operation. A backfill intentionally processes a historical range or missing period, though it may use the same pipeline.

# Section 4 — Scalability and performance

## 34. What is scalability?

The ability to handle increased data volume, frequency or concurrency while meeting performance and reliability requirements.

## 35. What is vertical scaling?

Increasing resources on one machine, such as CPU, memory or storage.

## 36. What is horizontal scaling?

Adding more machines or workers to distribute work.

## 37. Vertical vs horizontal scaling?

Vertical scaling increases the capacity of one machine. Horizontal scaling adds more machines or workers and requires coordination between them.

## 38. What is throughput?

The amount of work processed in a given time, such as 50,000 records per minute.

## 39. What is latency?

The time taken for data or an event to become available or move through a system.

## 40. Can a pipeline have high throughput and high latency?

Yes. A large daily batch may process millions of records efficiently but make the results available only once per day.

## 41. What is partitioning?

Dividing a dataset into smaller logical parts, often using a field such as date or region, so storage and processing can be managed more efficiently.

## 42. What is partition pruning?

A query engine skips partitions that cannot contain the requested data, reducing unnecessary reads.

## 43. Does adding more workers always improve performance?

No. Bottlenecks may be caused by source limits, network, poor queries, skewed partitions or destination constraints. More workers can increase cost and contention.

## 44. What should you measure before optimizing a pipeline?

Input volume, runtime, CPU, memory, query duration, network use, queue backlog, retry rate, output counts and cost.

# Section 5 — Governance and security

## 45. What is data governance?

Policies, roles and processes that manage data ownership, meaning, quality, access, protection, lineage and retention.

## 46. What is a data owner?

A person or role accountable for a dataset or business data domain and its appropriate use.

## 47. What is a data steward?

A person or role that helps maintain data definitions, quality and consistent usage.

## 48. What is least privilege?

Giving each person or service only the permissions needed for its task.

## 49. Authentication vs authorization?

Authentication verifies identity. Authorization determines what that identity is allowed to access or do.

## 50. What is PII?

Personally identifiable information: information that can identify a person directly or indirectly, depending on the applicable definition and policy.

## 51. How should a pipeline protect sensitive data?

Use approved access controls, encryption, data minimization, masking where appropriate, secure secret storage, safe logging and approved retention/deletion procedures.

## 52. Why should passwords not be stored in source code?

Source code may be shared, committed or exposed. Secrets should be stored in an approved secret manager or protected configuration, and never printed in logs.

## 53. What is encryption in transit?

Protection of data while it moves between systems, commonly using TLS.

## 54. What is encryption at rest?

Protection of stored data, such as database files, disks and object storage.

## 55. What is a retention policy?

A rule defining how long data must or may be stored and how it is handled after that period.

## 56. Why is governance part of Data Engineering?

Pipelines copy, transform and expose data. Engineers must implement approved controls and make sure data is handled responsibly.

# Section 6 — Monitoring and operations

## 57. What is pipeline monitoring?

Observing pipeline execution and data behavior to detect failures, delays, unusual volumes and quality issues.

## 58. Logs vs metrics vs alerts?

- Logs: records of events.
- Metrics: numerical measurements.
- Alerts: notifications when a defined condition occurs.

## 59. What should a pipeline log?

Run ID, pipeline name, timestamps, status, source and destination identifiers, rows read/written/rejected, duration and safe error details. Do not log secrets.

## 60. What is data freshness?

How up-to-date the available data is relative to the expected update time or business cutoff.

## 61. What is reconciliation?

Comparing source and destination counts, keys, totals or other measures to detect missing or incorrect results.

## 62. What is an SLA?

A formal service-level agreement that defines service commitments between parties.

## 63. What is an SLO?

A service-level objective: a target for a service measurement.

## 64. What is an SLI?

A service-level indicator: the actual measurement used to evaluate the service.

## 65. What is a runbook?

An operational document containing instructions for monitoring, troubleshooting, recovery and escalation.

## 66. What should you do when the daily pipeline fails?

Identify impact and failed stage, inspect safe logs and metrics, classify the error, follow retry or correction procedures, verify whether writes were committed, recover safely, reconcile output and communicate status.

# Section 7 — Scenario-based questions

## 67. Your pipeline loaded the same batch twice. What would you investigate?

I would check whether the destination load is idempotent, whether stable keys and constraints exist, whether the batch was committed before the failure, and whether the checkpoint advanced at the correct time. I would quantify the impact and correct duplicates using an approved recovery procedure.

## 68. The pipeline completed but the report is missing 10% of the orders. What would you do?

Check source extraction filters, pagination, date boundaries, rejected rows, joins, incremental watermark and destination load. Compare source and target keys and counts. Correct the cause and backfill the missing range safely.

## 69. The payment API is unavailable. Should the pipeline mark every order as unpaid?

No. An unavailable API means payment status is unknown, not necessarily unpaid. Follow the business rule: preserve an unknown or pending state, retry safely, alert and avoid publishing misleading revenue.

## 70. The source adds a new column. What should happen?

Follow the schema-change policy. Determine whether the column is optional or required, check downstream compatibility, update mappings and tests, document the change and deploy safely.

## 71. A daily pipeline now takes four hours instead of one. What would you inspect?

Compare input volume, query plans, data skew, API latency, worker resources, memory, storage, network and recent code or schema changes. Measure before scaling.

## 72. A report is late but the pipeline status says SUCCESS. Why?

The job may have completed after the deadline. Execution success and business freshness are different. Monitor completion time against the agreed deadline.

## 73. A developer asks you to put an API key in a GitHub file for convenience. What do you say?

No. Use approved secret storage or protected environment configuration. If a real key has already been committed, treat it as exposed and follow the rotation and incident process.

## 74. The pipeline has 100 invalid records. Should you stop the whole batch?

It depends on the agreed quality policy and business impact. Some systems fail the batch for critical errors; others process valid rows and quarantine invalid rows. The behavior must be documented, monitored and approved.

## 75. A historical report needs to be rebuilt for the last 30 days. What is your approach?

Confirm the date range, source history, target behavior, downstream impact and business rules. Plan a controlled backfill with duplicate protection, progress tracking, validation and stakeholder communication.

# Section 8 — Mini-project interview

## 76. Explain your Pipeline Requirements Document project.

"I designed a requirements document for a daily TechStore sales pipeline. It combines order data from a database, payment information from an API and delivery updates from a file. I documented the business objective, stakeholders, source systems, destination fields, functional and non-functional requirements, validation rules, risks, security controls, monitoring and acceptance criteria."

## 77. Why did you document requirements before writing code?

Because the business definitions, source behavior, freshness, security and recovery requirements determine the technical design. Coding before clarifying them can produce an incorrect pipeline.

## 78. How did you decide the pipeline schedule?

The business requested the previous day's report by 7 AM. I would confirm source availability and processing duration, then agree a schedule with enough time for validation, retries and reporting.

## 79. How would you validate the final report?

Check required fields, valid values, duplicate keys, source and destination counts, important totals, date boundaries, rejected records and freshness.

## 80. What are the most important risks?

Source outages, API rate limits, duplicate records, late data, schema changes, invalid values, destination failure, sensitive data exposure and unclear business definitions.

## 81. What would you improve before production?

Confirm all business rules and owners, test realistic failure scenarios, establish safe incremental loading, configure monitoring and alerts, verify access controls, define recovery and backfill procedures, and obtain stakeholder approval.

# Section 9 — Short-answer revision

| Question | Short answer |
|---|---|
| Pipeline | Repeatable data processing flow |
| Source | Where data originates |
| Destination | Where data is delivered |
| Source profiling | Inspect source structure and behavior |
| Full load | Read all records |
| Incremental load | Read new or changed records |
| Batch | Process grouped records |
| Streaming | Process ongoing events |
| Idempotency | Safe repeated operation |
| Checkpoint | Saved safe progress |
| Retry | Try an operation again |
| Backfill | Process historical data |
| Fault tolerance | Continue or recover from failure |
| Scalability | Handle increased workload |
| Throughput | Work per unit time |
| Latency | Time until data is available |
| Governance | Rules and accountability for data |
| Least privilege | Minimum necessary access |
| Reconciliation | Compare expected and actual results |
| Runbook | Operational instructions |

# Section 10 — Interview practice checklist

- [ ] I can define a data pipeline in my own words.
- [ ] I can describe the pipeline lifecycle.
- [ ] I can gather business and technical requirements.
- [ ] I can compare full and incremental extraction.
- [ ] I can compare batch and streaming.
- [ ] I can explain idempotency with an example.
- [ ] I can explain checkpoints and retries.
- [ ] I can distinguish transient and permanent failures.
- [ ] I can explain reliability, availability and fault tolerance.
- [ ] I can compare vertical and horizontal scaling.
- [ ] I can explain throughput, latency and partitioning.
- [ ] I can describe governance and least privilege.
- [ ] I can explain logs, metrics, alerts and freshness.
- [ ] I can explain reconciliation and backfills.
- [ ] I can present the Pipeline Requirements Document project.

---

# Final interview advice

For a junior Data Engineer interview, a clear answer is better than a list of tool names.

Use this structure:

1. Define the concept simply.
2. Explain why it matters.
3. Give a realistic example.
4. Mention one limitation or failure case.
5. Explain how you would handle it.

**Phase 1 complete:** You now have the foundation to understand what a pipeline is, why organizations need it and what engineers must consider before operating one. Phase 2 begins with Python fundamentals.
