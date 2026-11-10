# Day 5: Data Pipeline Fundamentals

**Phase:** 1 — Data Engineering Foundations  
**Learning time:** 2–3 hours  
**Mini-project:** Pipeline Requirements Document  
**Audience:** Freshers, college students and aspiring junior Data Engineers

---

# Part 1 — Start with the business problem

## 1. What will you learn today?

By the end of this lesson, you should be able to:

1. Explain a data pipeline in simple words.
2. Identify the source, destination and processing steps.
3. Describe the complete pipeline lifecycle.
4. Gather business and technical requirements before writing code.
5. Explain reliability, scalability and fault tolerance with practical examples.
6. Recognize common pipeline failures and choose a sensible response.
7. Understand data governance, data ownership, privacy and security basics.
8. Write a clear Pipeline Requirements Document.
9. Explain your design decisions in a junior Data Engineer interview.

Do not worry if these words look difficult now. We will introduce each one with a small example before using it in a realistic scenario.

## 2. Begin with a familiar situation

Imagine that you work for an online shopping company called **TechStore**.

Every day, customers place orders through the company's website and mobile application.

The company has information in different places:

- The website stores order information in an application database.
- The payment provider sends payment updates.
- The delivery partner provides shipment updates.
- The customer support team maintains complaint records.
- The management team wants a daily sales report.

The manager asks:

> "Can you prepare one correct report every morning showing yesterday's orders, revenue, payments and delivery status?"

Think about what you need to do.

You cannot simply open one file because the information is stored in different systems. You need to collect it, check it, combine it, transform it and make it available to the reporting team.

That repeatable sequence is a **data pipeline**.

## 3. What is a data pipeline?

A data pipeline is a sequence of connected steps that moves data from a source to a destination and performs the required processing along the way.

In simple words:

**Source -> Read -> Check -> Transform -> Store -> Monitor**

Example:

`Orders database -> Extract orders -> Validate records -> Calculate sales -> Load warehouse -> Daily report`

A pipeline is not necessarily one Python script. It may contain:

- Database queries.
- Python programs.
- File transfers.
- API calls.
- Data validation rules.
- SQL transformations.
- Cloud storage.
- Workflow scheduling.
- Monitoring and alerting.

### 3.1 Understand each word

| Word | Simple meaning | TechStore example |
|---|---|---|
| Source | Where data comes from | Orders database |
| Extract | Read or collect data | Read yesterday's orders |
| Validate | Check whether data follows rules | Order ID must not be empty |
| Transform | Change data into a useful form | Calculate order amount |
| Load | Save data into the destination | Insert records into warehouse |
| Destination | Where processed data is stored | Sales reporting table |
| Monitor | Check whether the process worked | Alert if the load fails |

### 3.2 Why do companies need pipelines?

Without a pipeline, an employee may have to:

1. Download orders manually.
2. Download payment details.
3. Download shipment details.
4. Open spreadsheets.
5. Match records using order IDs.
6. Remove duplicates.
7. Calculate totals.
8. Prepare a report.
9. Repeat the same work tomorrow.

This is slow and error-prone.

A pipeline automates repeatable work. It can run every hour, every day or whenever a new event arrives.

### 3.3 A pipeline is not just movement

Consider these two examples.

**Example A — Simple movement**

`orders.csv -> sales_folder/orders.csv`

The file was copied. No business validation or transformation happened.

**Example B — Data pipeline**

`orders.csv -> read -> validate -> reject invalid rows -> calculate amount -> load sales table -> compare row counts`

Example B contains a defined processing workflow.

Copying a file can be one pipeline step, but a production pipeline usually needs more than copying.

### 3.4 When do we use a pipeline?

Use a pipeline when data must be collected or processed repeatedly and the result must be delivered reliably.

Common examples:

- Daily sales reporting.
- Customer activity analysis.
- Payment reconciliation.
- Inventory updates.
- Delivery tracking.
- Fraud detection data preparation.
- Data warehouse refreshes.
- Machine learning feature preparation.

### 3.5 Check your understanding

**Question:** A company downloads a CSV manually once a year for a one-time analysis. Is automation always necessary?

**Answer:** No. A pipeline is most useful when a process is repeatable, has operational requirements, or needs dependable delivery. A one-time manual task may not justify building an automated system.

**Question:** A script copies a file but never checks whether it is complete. Is that a reliable production pipeline?

**Answer:** Not by itself. It moves data, but it does not establish that the data is complete, correct or safely delivered.

---

# Part 2 — Understand the pipeline lifecycle

## 4. What is a pipeline lifecycle?

A lifecycle is the sequence of stages through which a pipeline is planned, built, released and operated.

A Data Engineer should not begin by immediately writing code. First, the engineer must understand the business requirement.

A useful lifecycle is:

`Understand -> Profile -> Design -> Build -> Test -> Deploy -> Monitor -> Improve`

We will understand each stage using the TechStore daily sales report.

## 5. Stage 1 — Understand the requirement

The manager says:

> "I need yesterday's sales report by 7:00 AM every day."

This sounds simple, but many questions are unanswered.

Ask:

1. What does "sales" mean?
2. Does sales mean orders placed or payments successfully collected?
3. Which time zone defines yesterday?
4. Should cancelled orders be included?
5. Should refunded orders reduce revenue?
6. What should happen if payment data is delayed?
7. Who receives the report?
8. What happens if the report is not ready by 7:00 AM?

These are requirement questions.

### 5.1 Business requirement vs technical requirement

A **business requirement** describes the outcome a business user needs.

A **technical requirement** describes how the system must behave to deliver that outcome.

| Business requirement | Technical requirement |
|---|---|
| Show yesterday's sales | Filter records using the agreed business date |
| Report ready by 7 AM | Complete the pipeline before 7 AM |
| Exclude cancelled orders | Apply an agreed order-status rule |
| Show payment status | Integrate the payment source |
| Report must be correct | Validate totals and reconcile records |

Do not confuse the requested outcome with the implementation.

### 5.2 Ask for definitions

Words such as sales, active customer, completed order and revenue can mean different things to different teams.

For example:

- The sales team may count orders when customers place them.
- The finance team may count revenue only after payment is captured.
- The operations team may count orders after dispatch.

A Data Engineer must get the definition from the responsible business owner and document it.

**Beginner rule:** Never silently invent a business definition.

## 6. Stage 2 — Identify and profile the sources

A source is the system or location from which the pipeline reads data.

TechStore has three initial sources:

| Source | Example fields | Purpose |
|---|---|---|
| Orders database | order_id, customer_id, order_time, status, amount | Order details |
| Payments API | payment_id, order_id, payment_status, paid_amount | Payment details |
| Delivery CSV | order_id, delivery_status, updated_at | Shipment status |

### 6.1 What does source profiling mean?

Source profiling means examining data to understand its structure, quality and behavior before designing the pipeline.

You should inspect:

- Column names.
- Data types.
- Number of rows.
- Missing values.
- Duplicate records.
- Unique identifiers.
- Date ranges.
- Update patterns.
- Late-arriving records.
- Source availability.
- Data volume.

### 6.2 A tiny source example

Suppose the orders source contains:

| order_id | customer_id | status | amount |
|---|---|---|---:|
| 1001 | C01 | PAID | 1200 |
| 1002 | C02 | CANCELLED | 800 |
| 1003 | C03 | PAID | 500 |
| 1003 | C03 | PAID | 500 |
| 1004 | C04 | PAID | NULL |

What should you notice?

1. Order 1003 appears twice.
2. Order 1004 has no amount.
3. The amount column needs a defined numeric type.
4. The business rule must explain whether CANCELLED orders count as sales.
5. We need to know whether order_id is unique in the source or whether duplicate events are expected.

Do not immediately delete every duplicate. First understand why it exists.

### 6.3 Source profiling with SQL

Assume a table named `orders` already exists.

**Task:** Count the number of rows.

```sql
SELECT COUNT(*) AS total_rows
FROM orders;
```

Possible output:

| total_rows |
|---:|
| 12500 |

**Explanation:**

- `SELECT` asks the database to return information.
- `COUNT(*)` counts rows.
- `AS total_rows` gives the result column a readable name.
- `FROM orders` identifies the source table.

**Task:** Find rows where the amount is missing.

```sql
SELECT *
FROM orders
WHERE amount IS NULL;
```

Possible output:

| order_id | customer_id | status | amount |
|---|---|---|---|
| 1004 | C04 | PAID | NULL |

**Important:** Use `IS NULL`, not `= NULL`, to check SQL NULL values.

**Task:** Count records by status.

```sql
SELECT status, COUNT(*) AS order_count
FROM orders
GROUP BY status;
```

Possible output:

| status | order_count |
|---|---:|
| PAID | 9800 |
| CANCELLED | 1500 |
| PENDING | 1200 |

The values above are an illustrative example. Actual output depends on the source data.

### 6.4 Questions to ask the source owner

- Is the source updated continuously or at fixed times?
- Can old records be updated?
- Can records be deleted?
- Is there a reliable updated_at column?
- Is there a unique key?
- Can we read historical records?
- Are there API rate limits?
- Can the source be unavailable during maintenance?
- Who can explain the meaning of each field?

## 7. Stage 3 — Design the pipeline

After understanding the business requirement and sources, decide how data will move.

A basic design should identify:

1. Sources.
2. Extraction method.
3. Processing rules.
4. Validation rules.
5. Destination.
6. Schedule or trigger.
7. Failure behavior.
8. Security controls.
9. Monitoring.
10. Ownership.

### 7.1 Full extraction

Full extraction reads the complete dataset each time.

Example:

`Read all 100,000 orders -> Process all 100,000 -> Load destination`

Advantages:

- Easy to understand.
- Useful for small datasets.
- Can rebuild a destination from source data.

Disadvantages:

- Reprocesses old records.
- Can take more time.
- Uses more source and processing resources.
- May increase cost.

### 7.2 Incremental extraction

Incremental extraction reads only new or changed records since a known point.

Example:

Yesterday, the source had 100,000 orders. Today, 2,000 new or changed records are available.

Instead of reading all 100,000 records, the pipeline reads the 2,000 relevant records.

A common approach uses an update timestamp called a **watermark**.

Example query:

```sql
SELECT order_id, customer_id, status, amount, updated_at
FROM orders
WHERE updated_at > '2026-11-09 00:00:00';
```

The timestamp is illustrative. A real pipeline must store and manage its last successful watermark safely.

**Important beginner warning:** A timestamp-only approach can miss records if timestamps are not reliable, records arrive late, or multiple rows share the same timestamp. Production designs may use overlap windows, composite cursors, source change tracking or reconciliation.

### 7.3 Batch processing

Batch processing handles a group of records together.

Example:

- Collect all orders for the previous day.
- Process them at 2:00 AM.
- Load the daily sales table.
- Publish the report by 7:00 AM.

Batch is useful when the business does not need every event immediately.

### 7.4 Streaming processing

Streaming processing handles events continuously or in small ongoing flows.

Example:

- A customer places an order.
- An order event is produced.
- A consumer receives the event.
- The system updates a near-real-time order dashboard.

Streaming is useful when low latency matters. It also introduces more operational complexity.

### 7.5 Choose based on the requirement

| Requirement | Possible approach |
|---|---|
| Daily report by morning | Scheduled batch |
| Dashboard updated within seconds | Streaming or frequent micro-batches |
| Small source table | Full load may be sufficient |
| Very large changing table | Incremental extraction |
| Rebuild destination after corruption | Controlled full reload |
| External API with strict rate limit | Incremental requests with rate-limit handling |

There is no single correct architecture for every company. Choose based on freshness, volume, cost, complexity and business impact.

## 8. Stage 4 — Build the pipeline in small steps

A pipeline should be divided into understandable units.

For example:

1. Read orders.
2. Read payment information.
3. Read delivery information.
4. Validate each source.
5. Match records using order_id.
6. Apply business rules.
7. Write the output.
8. Record processing statistics.

Do not begin with one giant script that performs every task at once.

### 8.1 Why separate steps?

Suppose the final report has an incorrect payment total.

If the pipeline has clear stages, you can inspect:

- Was the payment API response correct?
- Did extraction miss a page?
- Did validation reject valid payments?
- Did the join duplicate records?
- Did the aggregation use the correct field?

Clear stages make troubleshooting easier.

### 8.2 Example of a simple transformation

Input:

| order_id | quantity | unit_price |
|---|---:|---:|
| 1001 | 2 | 500 |
| 1002 | 1 | 800 |
| 1003 | 3 | 100 |

Business rule:

`order_total = quantity × unit_price`

Output:

| order_id | quantity | unit_price | order_total |
|---|---:|---:|---:|
| 1001 | 2 | 500 | 1000 |
| 1002 | 1 | 800 | 800 |
| 1003 | 3 | 100 | 300 |

The transformation is simple, but a production pipeline must also define what happens when quantity or price is missing, negative or invalid.

## 9. Stage 5 — Test the pipeline

Testing checks whether the pipeline behaves as expected.

A pipeline that works for one clean example may fail with real business data.

Test at least these situations:

| Test case | Example | Expected behavior |
|---|---|---|
| Normal input | Valid order | Process successfully |
| Empty input | No new orders | Complete safely and report zero rows |
| Missing value | Amount is NULL | Apply documented rule or reject |
| Duplicate | Same order appears twice | Avoid duplicate business effect |
| Invalid type | Amount contains text | Reject or quarantine |
| Late record | Yesterday's order arrives today | Handle according to late-data policy |
| Source unavailable | API timeout | Retry safely and alert if unresolved |
| Destination unavailable | Database is down | Do not report a successful load |
| Schema change | New source column | Follow schema-change policy |

### 9.1 Unit test vs integration test

A **unit test** checks one small piece of logic.

Example: Does the amount calculation return 1,000 for quantity 2 and price 500?

An **integration test** checks whether multiple components work together.

Example: Can the pipeline read from the database, transform the rows and write the results to the target?

### 9.2 Reconciliation

Reconciliation compares source and destination results to find unexpected differences.

Example:

| Measurement | Source | Destination |
|---|---:|---:|
| Valid orders | 950 | 950 |
| Total amount | 750000 | 750000 |

This looks consistent.

Now consider:

| Measurement | Source | Destination |
|---|---:|---:|
| Valid orders | 950 | 920 |
| Total amount | 750000 | 721000 |

The mismatch needs investigation.

A row count alone is not enough. A pipeline can have the same row count but incorrect values. Compare important totals, keys, date ranges and business measures too.

## 10. Stage 6 — Deploy the pipeline

Deployment means making a tested pipeline available in the environment where it will run.

Typical environments:

- Development: where engineers build and experiment.
- Testing or staging: where changes are checked before release.
- Production: where the pipeline serves real business needs.

A beginner should understand that running a notebook on a laptop is not the same as deploying a production pipeline.

Before deployment, confirm:

- Code is version-controlled.
- Configuration is separated from code.
- Credentials are not hard-coded.
- Required permissions are available.
- Dependencies are documented.
- Tests pass.
- Schedule and time zone are correct.
- Monitoring is configured.
- Recovery steps are documented.
- An owner is assigned.

## 11. Stage 7 — Monitor and improve

After deployment, the job is not finished.

A Data Engineer must check whether the pipeline continues to work correctly.

Useful monitoring information includes:

- Run ID.
- Pipeline name.
- Start time.
- End time.
- Duration.
- Status.
- Rows read.
- Rows processed.
- Rows rejected.
- Rows written.
- Error category.
- Source freshness.
- Destination freshness.

Example run summary:

| Run ID | Status | Rows read | Rows loaded | Rows rejected | Duration |
|---|---|---:|---:|---:|---:|
| RUN-001 | SUCCESS | 1000 | 995 | 5 | 42 sec |
| RUN-002 | FAILED | 1200 | 0 | 0 | 18 sec |
| RUN-003 | SUCCESS | 1200 | 1198 | 2 | 51 sec |

Do not treat a green status as proof that the data is correct. A job may finish successfully while producing incomplete or incorrect results. Monitor data quality as well as execution status.

---

# Part 3 — Reliability

## 12. What is reliability?

Reliability means a pipeline performs its intended work correctly and consistently over time, including when expected problems occur.

Imagine the sales pipeline runs every morning.

A reliable pipeline should:

- Read the intended date range.
- Avoid silently losing records.
- Avoid creating duplicate business results.
- Detect invalid data.
- Report failures clearly.
- Recover safely.
- Produce results that can be checked.

Reliability is not simply "the code did not crash."

### 12.1 Reliability vs successful execution

Suppose a job finishes without an error but loads only 700 of the expected 1,000 records.

The execution status may be SUCCESS, but the business result is incomplete.

A reliable design checks both:

1. Did the process execute?
2. Did it produce the expected result?

## 13. Idempotency

Idempotency is a property that makes repeating the same operation safe: running it again with the same logical input does not create additional unintended effects.

### 13.1 Simple real-life example

Imagine you press an elevator button five times.

You still want one elevator request, not five separate elevator trips.

A similar problem occurs in data pipelines.

Suppose a batch contains these orders:

| order_id | amount |
|---|---:|
| 1001 | 500 |
| 1002 | 700 |

The pipeline loads them. Then the pipeline crashes before recording that the batch completed.

When it retries, it may load the same orders again.

Without duplicate protection, the destination may contain:

| order_id | amount |
|---|---:|
| 1001 | 500 |
| 1002 | 700 |
| 1001 | 500 |
| 1002 | 700 |

Revenue is now overstated.

### 13.2 How can we prevent duplicate effects?

Possible techniques include:

- Stable business keys.
- Unique constraints.
- Upsert or merge operations.
- Batch identifiers.
- Deduplication rules.
- Atomic transactions.
- Processed-event tracking.

The right technique depends on the destination and business semantics.

### 13.3 Upsert in simple words

An upsert means:

- If the record does not exist, insert it.
- If the record already exists, update it according to the defined rule.

A database may support `INSERT ... ON CONFLICT` or `MERGE`, depending on the database system.

Do not assume that every SQL database uses exactly the same syntax.

### 13.4 Important warning

Idempotency does not mean "ignore every duplicate row."

Two rows can have the same customer ID and still represent two different purchases.

Choose a key that identifies the business event or record correctly. A customer ID alone is not a valid deduplication key for orders.

## 14. Checkpointing

A checkpoint is a saved record of processing progress.

Example:

A pipeline processes 10,000 records in groups of 1,000.

| Batch | Records | Status |
|---|---:|---|
| 1 | 1,000 | Complete |
| 2 | 1,000 | Complete |
| 3 | 1,000 | Complete |
| 4 | 1,000 | Failed |
| 5 | 1,000 | Not started |

A checkpoint can help the pipeline know which work was completed.

### 14.1 Why do we need checkpoints?

Without checkpoints, a failed job may need to start from the beginning.

With safe checkpoints, the pipeline can resume from an appropriate point.

### 14.2 What should a checkpoint store?

Depending on the design, it may store:

- Last successfully committed timestamp.
- Last processed source ID.
- Batch number.
- File name or offset.
- Partition identifier.
- Processing status.
- Run identifier.

### 14.3 Do not update the checkpoint too early

Suppose a pipeline reads 1,000 records and updates its checkpoint before writing them to the destination.

Then the destination write fails.

On the next run, the pipeline may skip those 1,000 records because the checkpoint says they were already processed.

A safe design coordinates checkpoint updates with successful destination commits or uses a recovery strategy that prevents data loss.

**Remember:** A checkpoint is not merely a timestamp written somewhere. It must represent progress that is safe to resume from.

## 15. Retries

A retry means attempting an operation again after a failure.

Example:

The payment API temporarily returns a server error.

The pipeline waits and tries again.

### 15.1 Transient vs permanent errors

| Error type | Example | Retry? |
|---|---|---|
| Transient | Temporary network timeout | Usually yes, with limits |
| Service unavailable | API temporarily unavailable | Usually yes, with backoff |
| Rate limited | API says too many requests | Follow retry guidance and wait |
| Invalid credentials | Authentication rejected | Not repeatedly; fix credentials |
| Invalid SQL | Syntax error | No; fix the query |
| Missing required field | Invalid source record | Correct or quarantine the data |

### 15.2 Exponential backoff

Exponential backoff increases the wait between retry attempts.

Illustrative schedule:

| Attempt | Wait |
|---|---:|
| 1 | 1 second |
| 2 | 2 seconds |
| 3 | 4 seconds |
| 4 | 8 seconds |

Real systems often add a maximum delay and random jitter to avoid many clients retrying at the same time.

### 15.3 Why not retry forever?

Repeated retries can:

- Waste resources.
- Increase costs.
- Overload a failing service.
- Delay other work.
- Hide a permanent problem.

Set a retry limit. After the limit is reached, mark the task as failed, preserve diagnostic information and alert the owner.

## 16. Fault tolerance

Fault tolerance is the ability of a system to continue operating or recover when one or more components fail.

A pipeline can encounter:

- Network failure.
- API outage.
- Database outage.
- Worker crash.
- Full disk.
- Corrupted file.
- Invalid data.
- Permission failure.

Fault tolerance is achieved through a combination of design choices, not one magic setting.

Examples:

- Durable storage for incoming files.
- Retry policies for temporary failures.
- Checkpoints for safe recovery.
- Replication or redundant components where appropriate.
- Dead-letter or quarantine handling for problematic records.
- Backups and restore procedures.
- Monitoring and alerting.
- Documented recovery steps.

### 16.1 Fault tolerance vs reliability vs availability

| Term | Meaning |
|---|---|
| Reliability | Does the system deliver correct results consistently? |
| Availability | Is the system accessible when users need it? |
| Fault tolerance | Can it continue or recover when components fail? |
| Recoverability | Can service and data be restored after a problem? |

These ideas are related, but they are not interchangeable.

## 17. Dead-letter and quarantine patterns

Sometimes a record cannot be processed safely.

Example:

| order_id | amount | issue |
|---|---:|---|
| 1001 | 500 | Valid |
| 1002 | ABC | Amount is not numeric |
| 1003 | 700 | Valid |

A sensible design may:

- Process valid rows.
- Send invalid rows to a quarantine location.
- Record why each row was rejected.
- Alert or report the rejection count.
- Allow corrected records to be replayed.

A dead-letter queue is commonly used in message-processing systems to hold messages that cannot be processed after defined attempts.

A quarantine table or rejected-record file may be used in batch pipelines.

Do not silently discard bad records.

---

# Part 4 — Scalability and performance

## 18. What is scalability?

Scalability is the ability of a system to handle increased workload while continuing to meet its performance and reliability requirements.

A pipeline that processes 1,000 records today may need to process 10 million records next year.

A scalable design considers growth in:

- Data volume.
- Number of sources.
- Number of users.
- Processing frequency.
- Number of concurrent jobs.
- Data retention.
- Geographic distribution.

### 18.1 Vertical scaling

Vertical scaling means increasing the resources of one machine.

Examples:

- More memory.
- More CPU.
- Faster storage.

It is often easier to understand and may be sufficient for smaller workloads.

Limitations include hardware limits and potentially higher costs.

### 18.2 Horizontal scaling

Horizontal scaling means adding more machines or workers.

Example:

Instead of one worker processing 1 million independent records, multiple workers process separate partitions.

Benefits may include greater throughput and resilience.

However, distributed processing adds coordination, network, partitioning and consistency concerns.

### 18.3 Vertical vs horizontal scaling

| Vertical scaling | Horizontal scaling |
|---|---|
| Add resources to one machine | Add more machines or workers |
| Often simpler to operate | Requires coordination |
| Has machine limits | Can support larger distributed workloads |
| Example: increase server memory | Example: add processing workers |

### 18.4 Do not scale before measuring

Before adding infrastructure, measure:

- Input volume.
- Processing duration.
- CPU and memory use.
- Database query time.
- Network throughput.
- Queue backlog.
- Cost per run.
- Failure rate.

A slow pipeline may be caused by a poor SQL query, an unnecessary full reload or an API rate limit. Adding machines may not solve the actual problem.

## 19. Partitioning

Partitioning divides a large dataset into smaller logical parts.

Example: store orders by order date.

```text
orders/
  order_date=2026-11-08/
  order_date=2026-11-09/
  order_date=2026-11-10/
```

If a report needs only 9 November, a system that supports partition pruning may avoid reading unrelated date partitions.

Partitioning can improve manageability and performance, but poor partition choices can create too many tiny files or uneven workloads.

### 19.1 Choose a useful partition key

Possible partition keys:

- Order date.
- Event date.
- Region.
- Business unit.

Choose based on common filters, data distribution and query patterns.

Do not partition by a highly unique value such as order_id if that creates millions of tiny partitions.

## 20. Throughput and latency

**Throughput** describes how much work a system processes in a period.

Example: 50,000 records per minute.

**Latency** describes how long it takes for an event or record to move through a system or become available.

Example: an order becomes visible in a dashboard 5 seconds after creation.

| Term | Example question |
|---|---|
| Throughput | How many records can we process per minute? |
| Latency | How long does one record take to become available? |

A system may have high throughput but also high latency. For example, a large batch may process many records efficiently but publish results only once per day.

## 21. Batch size and parallel processing

A batch size is the number of records handled together in one processing group.

Small batches:

- May reduce memory use.
- May create more overhead.
- May increase the number of requests.

Large batches:

- May improve efficiency.
- May consume more memory.
- May increase the cost of retrying a failed batch.

The right size should be measured, not guessed.

Parallel processing allows independent work to happen at the same time.

Example:

- Worker A processes orders from region North.
- Worker B processes orders from region South.
- Worker C processes orders from region West.

Parallel processing helps only when tasks can be safely separated and the system has enough resources.

---

# Part 5 — Data governance and security

## 22. What is data governance?

Data governance is the set of policies, roles and processes used to manage data responsibly.

It answers questions such as:

- Who owns this data?
- What does each field mean?
- Who may access it?
- How accurate should it be?
- How long should it be retained?
- Where did it come from?
- Can it be shared?
- How should sensitive information be protected?

Governance is not only a documentation activity. It affects pipeline design and daily operations.

## 23. Data ownership and stewardship

A **data owner** is accountable for a dataset or business data domain.

A **data steward** helps maintain its meaning, definitions, quality and appropriate use.

A **Data Engineer** builds and operates the technical systems that move and process the data.

Example:

| Responsibility | Possible owner |
|---|---|
| Define what counts as recognized revenue | Finance data owner |
| Explain business meaning of payment_status | Payment data steward |
| Build payment ingestion pipeline | Data Engineer |
| Approve access to sensitive customer fields | Authorized data owner/security team |
| Investigate a failed scheduled load | Data Engineer / operations owner |

Actual responsibilities vary by organization. The key is to document who is accountable.

## 24. Data quality dimensions

A pipeline should define what good data means.

| Dimension | Simple meaning | Example check |
|---|---|---|
| Accuracy | Values represent reality | Payment amount matches payment source |
| Completeness | Required information exists | order_id is not missing |
| Consistency | Related systems agree | Same order status follows agreed mapping |
| Validity | Values follow rules | Amount is not negative |
| Uniqueness | Duplicate business records are controlled | One current order record per order key |
| Timeliness | Data arrives when needed | Daily load finishes before 7 AM |

A dataset can be complete but inaccurate. It can be accurate but late. Quality has multiple dimensions.

## 25. Data classification

Data classification groups data according to its sensitivity and handling requirements.

An example classification model:

| Category | Example | Handling |
|---|---|---|
| Public | Published product description | May be shared publicly |
| Internal | Internal pipeline status | Organization-only access |
| Confidential | Commercial contract details | Restricted access |
| Sensitive personal data | Customer identity or financial details | Strong access and protection controls |

Use the organization's approved classification policy. The labels above are illustrative, not a universal legal standard.

## 26. Personally identifiable information

Personally identifiable information, often shortened to PII, is information that can identify a person directly or indirectly.

Examples may include:

- Full name.
- Email address.
- Phone number.
- Government identifier.
- Home address.
- Account identifiers when linkable to an individual.

Not every organization or law defines PII in exactly the same way. Follow the applicable policy and law.

### 26.1 Why should a Data Engineer care?

A pipeline may copy personal data into:

- Raw storage.
- Temporary files.
- Logs.
- Data warehouse tables.
- Backups.
- Test environments.

Sensitive information can leak if engineers do not consider every place where data is stored or displayed.

### 26.2 Practical protection steps

- Collect only the fields required.
- Restrict access by role.
- Encrypt data in transit and at rest where required.
- Mask or tokenize fields when appropriate.
- Avoid exposing personal data in logs.
- Use approved retention and deletion processes.
- Protect backups and temporary files.
- Use synthetic or masked data in development where possible.

## 27. Authentication and authorization

These two terms are related but different.

**Authentication:** Who are you?

Example: a service proves its identity using an approved credential.

**Authorization:** What are you allowed to do?

Example: the service may read the orders table but cannot read payroll data.

| Authentication | Authorization |
|---|---|
| Verifies identity | Checks permissions |
| Login or service identity | Read, write, execute privileges |
| "Are you really this service?" | "May this service access this table?" |

## 28. Least privilege

Least privilege means giving a user or service only the permissions required for its task.

Example:

A daily sales pipeline needs to:

- Read order records.
- Read payment status.
- Write to a reporting schema.

It probably does not need:

- Permission to delete every production database.
- Access to employee payroll.
- Administrator privileges on the entire cloud account.

Smaller permissions reduce the impact of mistakes and compromised credentials.

## 29. Secrets management

A secret is sensitive information used to authenticate or authorize access.

Examples:

- Database password.
- API key.
- Access token.
- Private key.

Never commit real secrets to GitHub or place them directly in source code.

Bad example:

```python
database_password = "RealCompanyPassword123"
```

Better approach:

- Store secrets in an approved secret manager or protected environment configuration.
- Grant access only to the required service identity.
- Rotate credentials according to policy.
- Do not print secrets in logs.
- Never upload real credentials into teaching repositories.

Illustrative Python environment-variable access:

```python
import os

database_password = os.getenv("DB_PASSWORD")

if database_password is None:
    print("Database password is not configured")
else:
    print("Database password is available")
```

Possible output when the variable is absent:

```text
Database password is not configured
```

**Line-by-line explanation:**

- `import os` loads Python's operating-system utilities.
- `os.getenv("DB_PASSWORD")` reads an environment variable.
- If it is not configured, the result is `None`.
- The condition checks whether the value is missing.
- The program does not print the actual password.

This is a small illustration, not a complete production secrets-management solution.

## 30. Encryption in transit and at rest

**Encryption in transit** protects data while it moves between systems, commonly using TLS.

**Encryption at rest** protects stored data, such as database files, disks or object storage.

Example:

`Application -> encrypted network connection -> database`

and

`database storage -> encrypted disk/object storage`

Encryption does not replace access control. A person with valid access may still be able to read data. Use multiple security controls together.

## 31. Retention and deletion

Data should not automatically be stored forever.

A retention policy defines how long information must or may be kept.

Consider:

- Business needs.
- Legal and regulatory requirements.
- Contractual commitments.
- Storage costs.
- Backup retention.
- Deletion obligations.
- Audit requirements.

A pipeline may need to remove expired data or support deletion requests through approved processes.

Never invent a retention period. Get it from the responsible policy owner.

---

# Part 6 — Operational responsibilities

## 32. What does a Data Engineer do after building a pipeline?

A junior Data Engineer may help with:

1. Monitoring scheduled jobs.
2. Investigating failed runs.
3. Checking source and target counts.
4. Reviewing rejected records.
5. Fixing transformation bugs.
6. Supporting backfills.
7. Updating documentation.
8. Managing code changes through review.
9. Following security and access policies.
10. Escalating incidents to the right owner.

The work is not finished when the code runs once.

## 33. Logs, metrics and alerts

These terms are often confused.

**Logs** are timestamped records of events.

Example:

```text
2026-11-10 02:00:01 INFO Pipeline started
2026-11-10 02:00:04 INFO Read 1000 rows
2026-11-10 02:00:09 INFO Loaded 995 rows
2026-11-10 02:00:10 WARNING Rejected 5 rows
2026-11-10 02:00:10 INFO Pipeline completed
```

**Metrics** are numerical measurements.

Examples:

- 1,000 rows read.
- 995 rows loaded.
- 5 rows rejected.
- 42 seconds duration.
- 3 retries.

**Alerts** notify a responsible person or system when an important condition occurs.

Examples:

- Pipeline failed.
- Data is late.
- Rejected row count exceeded the threshold.
- Source has not updated.
- Destination storage is nearly full.

### 33.1 What should not be logged?

Do not log:

- Passwords.
- API tokens.
- Private keys.
- Full payment-card details.
- Unnecessary personal data.
- Sensitive raw payloads.

Logs themselves must be protected and retained according to policy.

## 34. Service-level expectations

A service-level agreement or SLA is a formal commitment between a service provider and its customer or business stakeholder.

An internal service-level objective or SLO is a target used to measure service performance.

A service-level indicator or SLI is the measurement itself.

Example:

- SLI: percentage of daily pipeline runs completed before 7 AM.
- SLO: target 99% of runs complete before 7 AM over the agreed measurement period.
- SLA: a contractual or formal commitment that may include consequences if the service misses agreed terms.

Do not promise an SLA unless the organization has approved it.

## 35. Freshness

Freshness describes how up-to-date the available data is.

Example:

The sales dashboard displays data only up to 8:00 AM, but it is now 11:00 AM.

The dashboard may be available, but its data is three hours old.

A freshness rule might say:

`The previous business day's sales data must be available by 7:00 AM`

Freshness is a business requirement, not merely a technical metric.

## 36. Incident response

An incident is an event that disrupts or threatens the expected service.

Example:

The daily sales report is missing at 7:00 AM.

A sensible response:

1. Confirm the affected pipeline and business impact.
2. Check the latest run status.
3. Identify the failed stage.
4. Inspect safe logs and metrics.
5. Check whether the source or destination is available.
6. Avoid unsafe repeated writes.
7. Follow the recovery procedure.
8. Notify the responsible stakeholders.
9. Verify the recovered output.
10. Document the cause and prevention action.

Do not make a production change blindly just to turn the status green.

## 37. Backfill

A backfill is the controlled processing of historical data that was not previously loaded or must be rebuilt.

Example:

The sales pipeline failed for 9 November. After fixing the issue, the team runs the pipeline for that missing business date.

A backfill plan should define:

- Date range.
- Source availability.
- Target overwrite or merge behavior.
- Duplicate protection.
- Expected volume.
- Downstream impact.
- Validation checks.
- Approval requirements where applicable.

A backfill is not simply "run the job again." It must be safe for historical data.

## 38. Runbook

A runbook is a practical document that explains how to operate or recover a system.

A pipeline runbook may include:

- Pipeline purpose.
- Owner and escalation contact.
- Schedule and time zone.
- Source and target.
- Dashboard or monitoring location.
- Common errors.
- Safe retry instructions.
- Backfill procedure.
- Data validation steps.
- Rollback or recovery guidance.

A good runbook helps another engineer support the pipeline without relying on undocumented knowledge.

---

# Part 7 — Practical design exercise

## 39. Scenario: TechStore daily sales pipeline

You are a junior Data Engineer on the TechStore data team.

The manager needs a daily report containing:

- Order ID.
- Customer ID.
- Order date.
- Order status.
- Payment status.
- Paid amount.
- Delivery status.

The report must be ready by 7:00 AM for the previous business day.

### 39.1 Source inventory

| Source | System | Data |
|---|---|---|
| Orders | PostgreSQL database | Orders and customers |
| Payments | REST API | Payment status and paid amount |
| Deliveries | CSV file in object storage | Delivery updates |

### 39.2 Define the destination

For this learning exercise, use a reporting table called:

`daily_sales_report`

Suggested columns:

| Column | Meaning |
|---|---|
| order_id | Unique order identifier |
| customer_id | Customer identifier |
| order_date | Business order date |
| order_status | Current agreed order status |
| payment_status | Payment result |
| paid_amount | Amount successfully paid |
| delivery_status | Latest delivery status |
| processed_at | Time this pipeline processed the record |

### 39.3 Define business rules

Before coding, document these rules and confirm them with the business owner:

1. Cancelled orders are excluded from recognized sales.
2. Pending payments are not counted as paid revenue.
3. Refunded payments must be represented separately or subtracted according to the finance-approved definition.
4. An order with no payment response must not be silently treated as paid.
5. Delivery status is taken from the latest valid delivery update available by the reporting cutoff.
6. All date boundaries use the agreed business time zone.
7. Invalid records are retained in a controlled rejected-record location with a reason.

These are example rules for the exercise. A real company must confirm its own definitions.

### 39.4 Draw the pipeline

```text
Orders database
      |
      v
Extract yesterday's orders
      |
      +----------------------+
      |                      |
      v                      v
Payments API            Delivery CSV
      |                      |
      +----------+-----------+
                 |
                 v
          Validate sources
                 |
                 v
       Match records by order_id
                 |
                 v
        Apply business rules
                 |
                 v
       Build daily sales output
                 |
                 v
       Load reporting destination
                 |
                 v
       Reconcile and monitor
                 |
                 v
        Publish report status
```

### 39.5 Think about failure scenarios

| Failure | Risk | Planned response |
|---|---|---|
| Orders database unavailable | No order input | Retry within policy, then alert |
| Payments API timeout | Payment status incomplete | Retry safely; do not assume paid |
| Delivery file missing | Delivery status incomplete | Apply agreed missing-file policy |
| Duplicate order | Inflated sales | Enforce business key and safe merge |
| Invalid amount | Incorrect revenue | Quarantine and report |
| Destination unavailable | Report not loaded | Preserve batch and retry safely |
| Job finishes late | Report misses deadline | Alert and communicate impact |

### 39.6 Define acceptance criteria

Acceptance criteria describe how the business and engineering team will decide that the pipeline is ready.

Example:

- The pipeline processes the agreed business date.
- All required source systems are checked.
- Required fields are validated.
- Duplicate business effects are prevented.
- Invalid rows are counted and retained with reasons.
- Paid revenue follows the approved finance definition.
- Destination totals are reconciled.
- The run status is visible.
- Failures generate an alert.
- The report is available by 7:00 AM under the agreed operating conditions.

Avoid vague criteria such as "the pipeline should work properly."

---

# Part 8 — Mini-project: Pipeline Requirements Document

## 40. Project objective

Create a requirements document for a daily order and sales pipeline.

You are not required to build a production pipeline today. Your task is to think and document like a Data Engineer before implementation.

**Project name:** Pipeline Requirements Document

**Business scenario:** TechStore wants a daily sales report by 7:00 AM.

## 41. Step 1 — Write the business objective

Example:

"The purpose of this pipeline is to combine order, payment and delivery information into a trusted daily sales dataset so that the management team can review previous-day business performance."

A good objective explains:

- Who needs the data.
- What information is needed.
- Why it is needed.
- When it must be available.

## 42. Step 2 — Document stakeholders

| Stakeholder | Responsibility |
|---|---|
| Sales manager | Defines sales reporting needs |
| Finance owner | Defines recognized revenue and refund rules |
| Operations team | Defines delivery status meaning |
| Data Engineer | Designs and operates the pipeline |
| Data platform owner | Supports infrastructure and access |
| Security team | Defines access and protection requirements |

## 43. Step 3 — Document source systems

For each source, capture:

- System name.
- Business owner.
- Technical contact.
- Connection or file location.
- Data format.
- Expected fields.
- Expected volume.
- Update frequency.
- Historical availability.
- Access method.
- Known limitations.

Do not place actual passwords or API keys in the document.

## 44. Step 4 — Document destination requirements

Capture:

- Destination name.
- Storage platform.
- Table or file structure.
- Required columns.
- Data types.
- Primary business key.
- Retention requirements.
- Downstream consumers.
- Access restrictions.

## 45. Step 5 — Document functional requirements

Functional requirements describe what the pipeline must do.

Example:

| ID | Requirement |
|---|---|
| FR-01 | Read orders for the previous business date |
| FR-02 | Retrieve payment information for the selected orders |
| FR-03 | Read the latest valid delivery updates |
| FR-04 | Validate required identifiers and amounts |
| FR-05 | Apply approved order and payment business rules |
| FR-06 | Load the reporting destination |
| FR-07 | Record processed, rejected and loaded row counts |
| FR-08 | Reconcile important source and destination totals |

## 46. Step 6 — Document non-functional requirements

Non-functional requirements describe how well the pipeline must operate.

| Area | Example requirement |
|---|---|
| Freshness | Report ready by 7:00 AM |
| Reliability | Failed runs must be detectable and recoverable |
| Performance | Complete within the agreed processing window |
| Scalability | Support expected data growth |
| Security | Use approved service identities and restricted access |
| Privacy | Do not expose unnecessary personal data |
| Maintainability | Keep code and configuration manageable |
| Observability | Record run status, counts and duration |
| Recovery | Document retry and backfill procedures |

The actual targets must be agreed with stakeholders and measured.

## 47. Step 7 — Document assumptions and dependencies

Examples:

**Assumptions**

- The orders source has a reliable order identifier.
- Finance will provide the recognized-revenue definition.
- Business date and time zone will be agreed.
- The delivery file has a documented schema.

**Dependencies**

- Database read access.
- Payment API access.
- Object storage access.
- Reporting destination availability.
- Approved credentials.
- Business owner review of the output.

An assumption is something believed to be true but not yet fully confirmed. A dependency is something the project needs from another system, team or process.

## 48. Step 8 — Document risks

| Risk | Impact | Mitigation |
|---|---|---|
| Payment API delay | Incomplete payment status | Retry and alert; do not assume payment |
| Duplicate source events | Inflated totals | Stable key and idempotent loading |
| Schema change | Extraction or transformation failure | Schema checks and change process |
| Late delivery updates | Outdated status | Define cutoff and late-data policy |
| Sensitive data exposure | Privacy/security incident | Least privilege and protected storage |
| Historical source data unavailable | Backfill impossible | Confirm retention and history access early |

## 49. Step 9 — Document monitoring and support

Define:

- What counts as success?
- What counts as failure?
- Which metrics are required?
- Which errors trigger alerts?
- Who receives alerts?
- Who owns recovery?
- How are backfills approved?
- How is a recovered run verified?

## 50. Step 10 — Review before implementation

Before development begins, review the document with:

1. Business owner.
2. Source-system owner.
3. Destination owner.
4. Security or platform representative.
5. Data Engineering lead.

A requirement that is unclear should be marked as an open question, not quietly guessed.

---

# Part 9 — Beginner exercises

## 51. Exercise 1 — Identify pipeline components

Scenario:

A hospital collects appointment data from an application database, validates patient appointment dates and loads daily appointment counts into a reporting warehouse.

Identify:

1. Source.
2. Extraction step.
3. Validation step.
4. Transformation.
5. Destination.
6. Business consumer.

**Suggested answer:**

- Source: application database.
- Extraction: read appointment records.
- Validation: check required IDs and valid appointment dates.
- Transformation: group appointments by date or department.
- Destination: reporting warehouse.
- Consumer: authorized hospital management or operations team.

Patient information is sensitive. The actual design must follow the hospital's privacy, security and access policies.

## 52. Exercise 2 — Choose the right processing pattern

| Scenario | Suitable starting point |
|---|---|
| Daily report of yesterday's orders | Batch |
| Live monitoring of machine events | Streaming or micro-batching |
| Small reference table refreshed weekly | Full load may be suitable |
| Large order table with frequent changes | Incremental extraction |
| Rebuild a corrupted reporting table | Controlled full reload |

Explain your reason. Do not memorize the table without understanding freshness, volume and complexity.

## 53. Exercise 3 — Find the reliability problem

A pipeline does this:

1. Reads 5,000 orders.
2. Inserts them into the destination.
3. Crashes before recording batch completion.
4. Runs again and inserts the same 5,000 orders.

**Question:** What is the risk?

**Answer:** Duplicate business records or inflated totals.

**Possible improvement:** Use a stable business key, a duplicate-safe merge or upsert, transaction boundaries and a checkpoint that advances only after a safe commit.

## 54. Exercise 4 — Find the security problem

A developer stores the database password directly in a Python file and commits it to a public repository.

**Problems:**

- The credential may be exposed.
- Other people may use it.
- The password may remain in Git history even after the line is deleted.

**Correct response:**

- Treat the credential as compromised.
- Notify the responsible owner.
- Rotate or revoke it.
- Remove exposure using the organization's approved process.
- Move secrets into approved secret storage.
- Review repository access and logs.

Deleting the visible line alone is not enough.

## 55. Exercise 5 — Write acceptance criteria

Requirement:

"The daily report should be accurate."

Rewrite it as measurable checks.

**Possible answer:**

- Required order IDs are present.
- Invalid amounts are rejected or quarantined.
- Duplicate order effects are prevented.
- Recognized revenue follows the approved finance definition.
- Source and destination counts are reconciled.
- Important totals are checked.
- The run status is recorded.
- The report is ready by the agreed deadline.

---

# Part 10 — Common beginner mistakes

## 56. Mistake: Starting with a tool instead of a requirement

**Incorrect thinking:** "We should use Kafka because it is popular."

**Better thinking:** "Does the business need continuous event processing? What freshness is required? What volume is expected? Can a batch process meet the need?"

Choose technology after understanding the requirement.

## 57. Mistake: Assuming successful execution means correct data

A pipeline can complete while missing records or applying the wrong business definition.

**Correction:** Check data quality, row counts, business totals, freshness and reconciliation.

## 58. Mistake: Retrying every error

Invalid credentials and invalid SQL are not fixed by repeated retries.

**Correction:** Classify failures. Retry transient failures within limits; route permanent failures for correction.

## 59. Mistake: Updating checkpoints before data is committed

This can cause skipped records after a failure.

**Correction:** Design checkpoint progression around safely committed work.

## 60. Mistake: Removing duplicates using the wrong key

Two purchases by the same customer are not necessarily duplicates.

**Correction:** Identify the business event key and understand source behavior.

## 61. Mistake: Ignoring late-arriving data

A record can arrive after the daily pipeline has already run.

**Correction:** Define late-data handling, overlap windows, correction loads or backfills.

## 62. Mistake: Logging secrets

Logs may be accessible to more people and systems than the original credential store.

**Correction:** Mask sensitive values and log safe error details only.

## 63. Mistake: Treating governance as someone else's work

A Data Engineer decides where data is copied, who can access it and how long it is retained.

**Correction:** Work with owners and security teams; apply approved rules in the pipeline.

## 64. Mistake: No owner or runbook

A pipeline fails at 2 AM and nobody knows who should investigate.

**Correction:** Document ownership, alert routing, recovery steps and escalation.

---

# Part 11 — Interview preparation inside the class

## 65. Explain a data pipeline in an interview

**Question:** What is a data pipeline?

**Beginner answer:**

A data pipeline is a repeatable process that collects data from one or more sources, validates and transforms it, and loads it into a destination for reporting or other use.

**Practical example:**

In an e-commerce company, a pipeline can collect orders from a database, payment details from an API and delivery updates from files. It validates and combines them and loads a daily sales dataset into a warehouse.

**Follow-up:** Is a pipeline only ETL?

**Answer:** No. A pipeline may use ETL, ELT, streaming, batch processing, file transfer or other patterns. The design depends on the requirement.

## 66. Explain how you would design a daily sales pipeline

Use this sequence:

1. Clarify the report and business definitions.
2. Identify the source systems and owners.
3. Profile source schemas and data quality.
4. Agree the business date and reporting cutoff.
5. Choose full or incremental extraction.
6. Define transformations and validation rules.
7. Design duplicate-safe loading and checkpointing.
8. Define security and access.
9. Test normal and failure cases.
10. Deploy with monitoring, alerting and a runbook.
11. Reconcile the output with source information.

Do not jump directly to a tool name.

## 67. Explain what happens when a pipeline fails

A strong answer:

"I first identify the failed stage and business impact. I inspect the run status, metrics and safe logs. I classify the error as transient or permanent. If it is transient, I follow the retry policy. If it is a data or code issue, I correct it and validate the affected batch. Before replaying, I check whether any destination writes were committed so that the retry does not create duplicates. I then verify the recovered output and communicate the result."

---

# Part 12 — Final revision

## 68. Key terms

| Term | Remember it as |
|---|---|
| Data pipeline | Connected steps that deliver data |
| Lifecycle | Plan, build, test, deploy and operate |
| Source profiling | Understand data before processing |
| Full extraction | Read the complete dataset |
| Incremental extraction | Read new or changed data |
| Batch | Process a group of records |
| Streaming | Process events continuously or as they arrive |
| Reliability | Consistent and correct results |
| Idempotency | Safe repeat of the same logical operation |
| Checkpoint | Saved safe processing progress |
| Retry | Attempt an operation again |
| Fault tolerance | Continue or recover when something fails |
| Scalability | Handle increased workload |
| Throughput | Work processed per unit of time |
| Latency | Time taken for data to become available |
| Governance | Rules, ownership and responsible data management |
| Least privilege | Only the permissions needed |
| Reconciliation | Compare source and destination results |
| Backfill | Process historical data |
| Runbook | Operational instructions |
| Freshness | How up-to-date the data is |

## 69. Final knowledge check

Try answering without looking at the notes.

1. What is a data pipeline?
2. Why should requirements be collected before coding?
3. What is source profiling?
4. What is the difference between full and incremental extraction?
5. When would you choose batch over streaming?
6. What does reliability mean?
7. How does idempotency prevent duplicate effects?
8. What is a checkpoint?
9. Why should checkpoints not advance before a safe commit?
10. What is the difference between retry and recovery?
11. What is fault tolerance?
12. What is the difference between vertical and horizontal scaling?
13. What is throughput?
14. What is latency?
15. Why is partitioning used?
16. What is data governance?
17. What is least privilege?
18. Why should passwords not be committed to Git?
19. What is data reconciliation?
20. What should a pipeline runbook contain?

## 70. Day 5 completion checklist

- [ ] I can explain a data pipeline without memorizing a definition.
- [ ] I can identify sources, transformations and destinations.
- [ ] I can explain each pipeline lifecycle stage.
- [ ] I can ask useful requirement questions.
- [ ] I can compare batch and streaming.
- [ ] I understand full and incremental extraction.
- [ ] I can explain reliability and idempotency.
- [ ] I understand checkpoints and safe retries.
- [ ] I can explain fault tolerance and scalability.
- [ ] I understand basic governance and security.
- [ ] I can identify useful pipeline metrics.
- [ ] I can create a Pipeline Requirements Document.
- [ ] I can explain my design in an interview.

---

# Day 5 outcome

You have completed the Data Engineering Foundations phase.

You should now understand the thinking that comes before pipeline implementation: business requirements, sources, design choices, quality rules, failure handling, security, monitoring and operational ownership.

**Next phase:** Phase 2 — Python for Data Engineering.

In Day 6, we begin Python from the beginning and connect programming concepts to data engineering tasks. We will not assume that you already know Python.
