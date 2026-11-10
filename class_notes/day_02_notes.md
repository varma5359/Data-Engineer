# Day 2: Data Processing Fundamentals

**Phase:** 1 — Data Engineering Foundations  
**Learning time:** 2–3 hours  
**Level:** Absolute beginner  
**Mini-project:** Data Processing Comparison Lab

> **How to study this lesson:** Imagine that you are working for the TechMart online retail company. We will use the same business throughout the lesson. First understand the business situation, then learn the technical term. You do not need to memorize every tool name today.

---

## 1. What you will learn today

By the end of this lesson, you should be able to:

1. Explain what data processing means.
2. Understand why operational and analytical workloads have different needs.
3. Explain OLTP and OLAP using simple business examples.
4. Compare transactional queries with analytical queries.
5. Explain batch processing step by step.
6. Explain stream processing step by step.
7. Distinguish real-time, near-real-time and batch requirements.
8. Understand event-driven architecture.
9. Identify producers, events, brokers and consumers.
10. Understand duplicate delivery, ordering and late-arriving events.
11. Choose a suitable processing pattern for a business requirement.
12. Complete the Data Processing Comparison Lab and defend your choices.

### Suggested class timing

| Lesson area | Suggested time |
|---|---:|
| Meaning of data processing | 10 minutes |
| OLTP and OLAP | 30 minutes |
| Batch processing | 20 minutes |
| Stream processing | 20 minutes |
| Real-time and near-real-time | 15 minutes |
| Event-driven architecture | 25 minutes |
| Delivery, ordering and recovery | 20 minutes |
| Mini-project and revision | 30 minutes |

---

## 2. First understand: what is data processing?

### 2.1 A simple example

A customer places an order on TechMart's website.

The application records:

| Field | Value |
|---|---|
| order_id | O201 |
| customer_id | C101 |
| product | Laptop |
| quantity | 1 |
| amount | 55000 |
| status | CONFIRMED |

This is a recorded order. But the company may need to do more work with it.

For example:

- Check whether the order has a valid customer.
- Check whether the quantity is positive.
- Connect it to the payment record.
- Calculate sales by product.
- Update inventory.
- Make the order available to a dashboard.

The work of reading, checking, changing, combining, calculating or delivering data is called **data processing**.

### 2.2 Beginner definition

**Data processing is the set of operations performed on data to make it useful for a particular task.**

Examples of processing:

| Operation | What it means | TechMart example |
|---|---|---|
| Read | Obtain data | Read new orders |
| Filter | Keep records that meet a condition | Keep paid orders |
| Validate | Check expected rules | Check order ID is present |
| Transform | Change the representation | Convert date text to a date |
| Join | Connect related records | Match order with payment |
| Aggregate | Calculate a summary | Total sales by day |
| Store | Save the result | Write prepared records to a warehouse |
| Deliver | Make the result available | Refresh a dashboard table |

### 2.3 Why do we need different processing methods?

Not every business question needs the same response time.

Consider these questions:

1. Can this customer place an order right now?
2. How much revenue did we earn last financial year?
3. Should a suspicious payment be flagged immediately?
4. Can management receive a stock report every morning?

Question 1 needs a quick operational transaction. Question 2 needs analysis across historical records. Question 3 may need low-latency event processing. Question 4 may be handled by a scheduled batch.

The business requirement determines the processing design.

### 2.4 A simple decision picture

```
What does the business need?
          |
          +--> Complete an operation now?
          |        |
          |        +--> Operational transaction (OLTP)
          |
          +--> Analyze many records?
          |        |
          |        +--> Analytical workload (OLAP)
          |
          +--> Process a group on a schedule?
          |        |
          |        +--> Batch processing
          |
          +--> React as events arrive?
                   |
                   +--> Streaming processing
```

OLTP/OLAP describe workload purposes. Batch/streaming describe processing patterns. They are related design concepts, but they are not interchangeable terms.

---

## 3. Understand OLTP from the beginning

### 3.1 What does OLTP stand for?

OLTP means **Online Transaction Processing**.

Break the name into parts:

- Online: the application is actively being used.
- Transaction: one logical business operation.
- Processing: the system performs that operation.

An OLTP system supports the daily activities of a business.

Examples:

- A customer places an order.
- A payment is recorded.
- An employee updates an address.
- A warehouse reserves one item.
- A delivery status changes.

### 3.2 What is a transaction?

A transaction is a logical unit of work.

Imagine that a customer purchases one laptop.

The application may need to:

1. Create an order.
2. Record the order items.
3. Reserve inventory.
4. Record or confirm payment.

These actions are related. The system must manage them carefully so that it does not leave the business in an inconsistent state.

A transaction is not necessarily one SQL statement. It may contain multiple statements managed as one logical unit.

### 3.3 What happens when a customer places an order?

Imagine customer C101 places order O201.

Before confirming the order, the application may check:

- Does the customer exist?
- Is the product available?
- Is the requested quantity valid?
- Can the payment be authorized?

The application then records the appropriate changes.

This is operational work because the system is helping the business complete an action.

### 3.4 Typical OLTP characteristics

| Characteristic | Beginner explanation |
|---|---|
| Many short transactions | Many customers perform small operations |
| Frequent writes | New orders and updates are recorded |
| Current records | The application needs current operational state |
| Concurrent users | Many users may act at the same time |
| Transaction correctness | Partial or conflicting changes must be controlled |
| Point lookups | Find one order using its ID |
| Fast response | The customer should not wait unnecessarily |

These are common patterns, not rules that every OLTP database must follow identically.

### 3.5 Example: an order lookup

A customer asks, "What is the status of order O201?"

The application needs a small number of records related to O201. It does not need to calculate every order placed during the last five years.

This is a point lookup: retrieving a record using a known identifier.

### 3.6 Example: updating an order status

Suppose order O201 changes from CONFIRMED to SHIPPED.

The application updates the current order status. It may also record a status-history event for audit and tracking.

The update must be controlled so that two processes do not unintentionally overwrite one another.

### 3.7 Why should an OLTP system avoid unnecessary heavy analysis?

A customer-facing database has work to do continuously.

If a large report scans huge amounts of data while customers are placing orders, both workloads may compete for CPU, memory, storage throughput and database connections.

The customer may experience slower responses.

This is one reason organizations often move analytical workloads to a separate platform.

---

## 4. Understand OLAP from the beginning

### 4.1 What does OLAP stand for?

OLAP means **Online Analytical Processing**.

It supports questions about business performance, trends and patterns.

Examples:

- What was monthly revenue by region?
- Which product category grew fastest?
- How many customers purchased again within 90 days?
- Which delivery partner had the most late shipments?
- How did sales compare with the previous year?

### 4.2 What makes an analytical question different?

Consider the question:

"Show monthly paid sales by product category for the previous 12 months."

To answer it, a system may need to:

1. Read many order records.
2. Read payment information.
3. Connect products to categories.
4. Apply the business definition of paid sales.
5. Group the results by month and category.
6. Calculate totals.
7. Return a summarized result.

This is broader than retrieving one order.

### 4.3 Typical OLAP characteristics

| Characteristic | Beginner explanation |
|---|---|
| Large reads | The query may examine many records |
| Aggregations | Calculate totals, averages and counts |
| Joins | Combine information from different datasets |
| Historical analysis | Compare data across days, months or years |
| Read-heavy workload | Many operations analyze rather than update individual transactions |
| Analytical organization | Data is arranged to support reporting and analysis |

### 4.4 Example of an analytical result

Suppose the warehouse contains these paid orders:

| order_id | order_date | category | paid_amount |
|---|---|---|---:|
| O201 | 2026-11-01 | Electronics | 55000 |
| O202 | 2026-11-01 | Accessories | 1200 |
| O203 | 2026-11-02 | Electronics | 42000 |
| O204 | 2026-11-02 | Home | 3000 |

Management asks for total paid amount by category.

The analytical result is:

| category | total_paid_amount |
|---|---:|
| Electronics | 97000 |
| Accessories | 1200 |
| Home | 3000 |

The system has grouped multiple records and calculated a summary.

### 4.5 Does OLAP mean only dashboards?

No. Dashboards are one consumer of analytical data.

Other consumers include:

- Business reports.
- Ad hoc SQL analysis.
- Data science workflows.
- Forecasting systems.
- Strategic planning.
- Regulatory reporting.

### 4.6 Does OLAP always require a separate product?

No. OLAP describes an analytical workload and approach. Organizations use different technologies depending on scale, architecture and business needs.

A small company may begin with one database. As workloads grow, separating operational and analytical processing may become beneficial.

---

## 5. OLTP vs OLAP — understand the difference properly

### 5.1 Business purpose

- OLTP helps the business perform its daily operations.
- OLAP helps the business understand its performance.

Think:

- OLTP: "Place this order."
- OLAP: "How many orders did we receive this quarter?"

### 5.2 Side-by-side comparison

| Dimension | OLTP | OLAP |
|---|---|---|
| Full form | Online Transaction Processing | Online Analytical Processing |
| Main goal | Run operations | Analyze operations |
| Typical question | What is the status of order O201? | What was monthly revenue? |
| Data focus | Current operational records | Historical and integrated records |
| Common activity | Insert, update, point lookup | Scan, join, group, aggregate |
| Query size | Often small and targeted | Often broad and complex |
| Users | Applications, customers, staff | Analysts, managers, data scientists |
| Performance concern | Transaction response and correctness | Analytical query performance |
| Example | Checkout process | Sales performance report |

### 5.3 A common interview trap

**Question:** "Is OLTP the same thing as a database and OLAP the same thing as a data warehouse?"

**Answer:** No. OLTP and OLAP describe workload patterns and processing purposes. Databases and warehouses are systems that may support those workloads.

A relational database product can support different workloads, although its design and configuration may differ.

### 5.4 Why do companies often separate them?

Imagine a busy shopping application with 10,000 customers placing orders.

At the same time, a manager runs a report that scans hundreds of millions of historical order lines.

Possible effects when both workloads compete:

- Customer queries may become slower.
- CPU and memory resources may be heavily used.
- Database connections may become busy.
- Storage reads may compete.
- Performance may become unpredictable.

A common design sends data to an analytical environment where large reports can run without directly competing with checkout transactions.

### 5.5 Why not copy every record immediately?

Copying data introduces its own responsibilities:

- Data may arrive late.
- Some records may be updated after the first copy.
- Duplicate records may arrive.
- Source and destination schemas may differ.
- The destination may temporarily fail.

Separating workloads is useful, but the data movement process must also be reliable.

### 5.6 Quick understanding check

For each task, decide whether it is mainly operational or analytical.

| Task | Main workload | Reason |
|---|---|---|
| Create a customer account | OLTP | Records a business operation |
| Update payment status | OLTP | Changes current transaction state |
| Calculate yearly revenue | OLAP | Aggregates historical records |
| Find one order by ID | OLTP | Targeted operational lookup |
| Compare sales by region for 3 years | OLAP | Historical comparison |
| Count today's orders for a dashboard | Usually OLAP/reporting | Produces a business summary |

A task's implementation may vary. The important part is understanding its purpose and workload.

---

## 6. Batch processing — start with a daily example

### 6.1 What is batch processing?

**Batch processing means collecting a defined group of records and processing that group together.**

A batch has a boundary. For example:

- All orders received yesterday.
- All files delivered by a partner today.
- All transactions for a monthly billing period.
- All records in a particular input file.

### 6.2 Imagine a daily sales report

TechMart wants the previous day's sales report by 7 AM.

The company decides to process orders once every night.

Example schedule:

- 12:00 AM: Previous business day ends.
- 12:15 AM: Extract order and payment data.
- 12:30 AM: Validate the received records.
- 12:45 AM: Transform and aggregate data.
- 1:00 AM: Load the warehouse summary.
- 1:05 AM: Record the result and notify the team.

The times are illustrative. A real schedule depends on source availability and the reporting deadline.

### 6.3 Batch processing flow

```
Source records
      |
      v
Collect a defined batch
      |
      v
Validate the batch
      |
      v
Transform the records
      |
      v
Load the destination
      |
      v
Record status and metrics
```

### 6.4 What is a batch boundary?

A batch boundary tells us which records belong to one processing run.

Possible boundaries:

- Date: orders from 10 November.
- Time window: events from 10:00 AM to 11:00 AM.
- File: partner_delivery_2026_11_10.csv.
- Sequence range: records with source IDs 1000 to 1999.
- Partition: a specific date or region.

A clear boundary helps engineers count, validate and rerun the same set of records.

### 6.5 Batch example with a small dataset

Suppose a file contains:

| order_id | amount | status |
|---|---:|---|
| O301 | 1000 | PAID |
| O302 | 500 | PENDING |
| O303 | 2000 | PAID |
| O304 | 700 | PAID |

The business wants total paid amount.

Step 1: Read the batch — four records.

Step 2: Validate the records — check that order_id, amount and status are present and valid.

Step 3: Filter paid records — O301, O303 and O304.

Step 4: Add paid amounts — 1000 + 2000 + 700.

Step 5: Store the result.

Expected result:

| metric | value |
|---|---:|
| input_records | 4 |
| paid_records | 3 |
| total_paid_amount | 3700 |

### 6.6 Explain the result

- input_records = 4 because four rows arrived.
- paid_records = 3 because three rows have status PAID.
- total_paid_amount = 3700 because only paid amounts were added.

Notice that the pending amount of 500 is not included.

A real revenue metric may need additional rules for refunds, taxes, cancellations and settlement. The example uses a deliberately simple definition.

### 6.7 Advantages of batch processing

1. It is often easier to understand and operate.
2. It can process large groups efficiently.
3. A bounded batch can be validated before loading.
4. Failed batches can be retried or rerun when designed correctly.
5. It may cost less than maintaining a continuous streaming system.
6. It is suitable when the business does not need immediate results.

### 6.8 Limitations of batch processing

1. Results are not available until processing runs.
2. A failed batch may delay reports.
3. Very large batches may require partitioning.
4. Late-arriving records need a correction strategy.
5. Reprocessing must avoid duplicate output.
6. A daily schedule may not meet a seconds-level requirement.

### 6.9 When should we use batch processing?

Use batch when:

- A daily or hourly result is sufficient.
- Data arrives as files or scheduled extracts.
- The business works with reporting periods.
- Processing can happen after the events occur.
- Simpler operations are preferred and meet the requirement.

Examples:

- Daily finance report.
- Monthly invoice calculation.
- Nightly product catalog refresh.
- Daily customer segmentation.
- Historical data migration.

### 6.10 Important batch terms

**Batch ID:** A unique identifier for one batch run.

**Batch window:** The time period covered by a batch.

**Input count:** Number of records received.

**Output count:** Number of records successfully written.

**Rejected count:** Number of records that failed defined checks.

**Rerun:** Execute a batch again.

**Backfill:** Process historical data for a previous period.

**Partition:** A logical subdivision of data, often based on date or another key.

These terms help engineers describe what was processed and investigate failures.

---

## 7. How to make batch processing safe

A batch job should not simply read data and insert it without checking what happened.

### 7.1 Record the batch identity

For each run, record information such as:

| Metadata | Example |
|---|---|
| batch_id | ORDERS_20261110_001 |
| source | order_database |
| business_date | 2026-11-10 |
| started_at | 2026-11-11 00:15 |
| input_count | 50000 |
| output_count | 49980 |
| rejected_count | 20 |
| status | COMPLETED_WITH_REJECTIONS |

These values are examples. A real pipeline should use the correct time zone and run metadata.

### 7.2 Why compare input and output counts?

Suppose 50,000 records were received but only 48,000 were loaded.

The difference is 2,000 records.

That difference may be expected if records were rejected under documented rules. Or it may indicate a processing bug.

The engineer must be able to explain the difference.

### 7.3 What is a safe rerun?

Suppose a batch loads 10,000 orders and then fails while writing the final status.

If the engineer simply reruns the insert, the same 10,000 orders may be inserted again.

A safe design may use:

- Stable business keys.
- Upsert or merge behavior where appropriate.
- A batch-control table.
- Staging before final publication.
- Transaction boundaries.
- A rule for detecting already-processed input.

The correct method depends on the destination and business requirements.

### 7.4 What is a backfill?

Suppose the pipeline was broken on Monday and Tuesday. On Wednesday, the team fixes the issue.

The team may need to process Monday and Tuesday's missing data. This is called a backfill.

A backfill should define:

- Which dates or partitions are missing?
- Are the source records still available?
- Will existing records be overwritten or merged?
- How will duplicates be prevented?
- How will totals be reconciled?

### 7.5 What about late-arriving records?

Suppose an order was created at 11:58 PM but reached the data platform at 12:20 AM.

If the batch for that business date already ran, the record may be missing from the first report.

The platform needs a late-arrival policy, such as:

- Reprocess a recent date window.
- Run a correction batch.
- Update the affected partition.
- Publish a revised report.

The correct policy depends on how late data can arrive and how quickly the report must be finalized.

---

## 8. Stream processing — understand continuous events

### 8.1 What is a stream?

A stream is a continuing sequence of records or events.

Imagine thousands of customers browsing TechMart:

- Customer C101 views a laptop.
- Customer C102 adds a phone to a cart.
- Customer C103 completes a payment.
- Customer C101 searches for headphones.

These events may arrive continuously.

### 8.2 What is stream processing?

**Stream processing means processing events continuously or in small increments as they arrive, rather than waiting for one large scheduled batch.**

The purpose is often to make information available with low delay.

### 8.3 Simple streaming flow

```
Application / Device
        |
        v
   Event Producer
        |
        v
  Event Transport
   (e.g. broker)
        |
        v
  Stream Consumer
        |
        v
 Process / Validate
        |
        v
Alert / Store / Update
```

### 8.4 Example: payment monitoring

A customer attempts a payment of ₹75,000.

The payment service produces an event:

| Field | Example value |
|---|---|
| event_id | E9001 |
| event_type | PaymentAttempted |
| payment_id | PAY9001 |
| customer_id | C101 |
| amount | 75000 |
| event_time | 2026-11-10 14:20:00 |

A fraud-monitoring consumer receives the event and evaluates it against defined rules or a model.

It may create an alert for investigation.

The event does not automatically prove fraud. It is information that a downstream system can evaluate.

### 8.5 What is an event?

An **event** is a record that something happened.

Examples:

- OrderCreated.
- PaymentSucceeded.
- PaymentFailed.
- ProductViewed.
- ShipmentDispatched.
- TemperatureThresholdExceeded.

An event usually describes an occurrence, including identifiers and a time.

### 8.6 What is a producer?

A **producer** is the system that creates and publishes events.

Example: the payment application publishes PaymentSucceeded after a payment succeeds.

### 8.7 What is a consumer?

A **consumer** is a system that reads and processes events.

Examples:

- Notification service.
- Fraud monitoring service.
- Inventory service.
- Analytics ingestion service.

One event may be useful to multiple consumers.

### 8.8 What is a broker?

A **message broker** or event broker is infrastructure that receives, stores or transports messages/events and makes them available to consumers.

Apache Kafka is a widely used event-streaming platform. At this stage, understand its role conceptually; implementation and architecture details come later.

A broker is not the whole data platform. Consumers, processing logic, storage, monitoring and governance are also needed.

### 8.9 What is a topic?

In Kafka-style terminology, a topic is a named stream of events.

Examples:

- orders
- payments
- shipments
- website_clicks

A topic helps organize related events so consumers can subscribe to the stream they need.

### 8.10 What is a partition?

A partition is an ordered portion of a topic's event log.

Partitions allow a topic to be distributed and processed in parallel. Ordering is generally guaranteed within a partition, not automatically across every partition.

The partitioning key therefore matters when related events must be processed in order.

### 8.11 What is an offset?

An offset identifies a position of a record within a partition.

A consumer can track its progress using offsets.

For example, if a consumer has processed through offset 120, it knows its position in that partition. The exact commit and recovery behavior depends on the consumer configuration and processing design.

### 8.12 What is a consumer group?

A consumer group is a set of consumers working together to read a topic. Partitions are assigned among group members.

This can allow work to be distributed. The number of consumers that actively read in parallel within one group is constrained by the number of partitions.

These are introductory concepts. You will study Kafka in greater detail later in the course.

---

## 9. Batch vs streaming — compare using business requirements

### 9.1 Main comparison

| Dimension | Batch | Streaming |
|---|---|---|
| Input | Defined group of records | Ongoing sequence of events |
| Processing | Scheduled or triggered | Continuous or incremental |
| Typical latency | Minutes to hours, depending on schedule | Seconds or lower when designed for it |
| Complexity | Often simpler | Often more operationally complex |
| Recovery | Rerun a batch or partition | Resume/replay from tracked progress |
| Example | Daily sales report | Payment-risk alert |
| Cost profile | Often runs at intervals | May require continuously running infrastructure |

These are common tendencies, not absolute guarantees. A poorly designed batch can be complex, and a small stream processor can be simple.

### 9.2 Scenario 1: Monthly payroll report

The company calculates payroll once per month.

Does it need every employee's payroll total to update within milliseconds?

Usually not. A controlled batch process may be suitable.

### 9.3 Scenario 2: Payment authorization

A customer waits for a payment result during checkout.

The system must respond quickly enough for the transaction to continue. A synchronous operational request or low-latency event-driven design may be appropriate, depending on the architecture.

Do not assume every payment flow is implemented as a streaming analytics job.

### 9.4 Scenario 3: Website click analytics

The company wants to capture user clicks continuously and study activity shortly afterward.

Streaming ingestion may be useful because events arrive throughout the day.

A batch process may still be used later to create daily summaries.

### 9.5 Can batch and streaming work together?

Yes. Many real platforms use both.

Example:

1. Website events are ingested continuously.
2. Events are stored in a raw event platform.
3. A stream processor produces quick operational metrics.
4. A scheduled batch job creates trusted daily aggregates.
5. Analysts use the warehouse tables for historical reporting.

Streaming and batch are not enemies. They can serve different needs in one architecture.

---

## 10. Real-time, near-real-time and batch

### 10.1 What does latency mean?

Latency is the time between an event or input becoming available and the result being delivered.

Example:

- A payment event occurs at 10:00:00.
- An alert appears at 10:00:02.
- The event-to-alert delay is approximately two seconds.

Latency should be measured using a clearly defined start and end point.

### 10.2 What is real-time processing?

Real-time processing aims to produce a result within a defined time constraint.

The important point is that the requirement must be measurable.

For example:

"The payment risk check must return a decision within 300 milliseconds for 99% of requests."

This is more useful than simply saying "make it real-time."

Some real-time systems have hard deadlines. Others use low-latency targets. The exact meaning depends on the application.

### 10.3 What is near-real-time processing?

Near-real-time processing makes data available after a small delay that the business considers acceptable.

Examples:

- Refresh a stock dashboard every five minutes.
- Update a delivery tracking page every minute.
- Make new orders visible to an operations dashboard within two minutes.

### 10.4 What is batch processing in this comparison?

Batch processing waits for a defined group or scheduled run.

Examples:

- Daily revenue report at 6 AM.
- Weekly customer summary.
- Monthly invoice processing.

### 10.5 Comparison table

| Pattern | Example delay | Example use case |
|---|---|---|
| Real-time / low-latency | Milliseconds or seconds, based on requirement | Immediate transaction decision |
| Near-real-time | Seconds or minutes | Frequently refreshed stock dashboard |
| Batch | Scheduled intervals | Daily or monthly report |

The time ranges are illustrative. Always obtain the business's actual target.

### 10.6 How should you ask for a latency requirement?

Ask the business:

1. How quickly must the result be available?
2. Is that deadline strict or approximate?
3. What percentage of requests must meet it?
4. What happens if the result is late?
5. Does every record need immediate processing?
6. Can a dashboard refresh every few minutes instead?
7. How much additional cost is justified by lower latency?

### 10.7 A common mistake

A manager says, "We need real-time sales."

Do not immediately select Kafka or another streaming technology.

First clarify:

- Does the manager need every sale visible within one second?
- Is a one-minute delay acceptable?
- Is the dashboard only used every morning?
- Does the system need an immediate business action, or just a frequently refreshed chart?

Technology should follow the requirement.

---

## 11. Event-driven architecture — learn the complete picture

### 11.1 What is event-driven architecture?

Event-driven architecture is a design in which systems publish events when something happens and other systems respond to those events.

Instead of one application directly coordinating every downstream task, it can publish an event for interested consumers.

### 11.2 TechMart order example

A customer successfully places order O401.

The order service publishes:

```json
{
  "event_id": "EVT401",
  "event_type": "OrderCreated",
  "order_id": "O401",
  "customer_id": "C101",
  "event_time": "2026-11-10T14:30:00Z"
}
```

This is an illustrative event payload.

### 11.3 Who does what?

**Order service — producer**

Creates the OrderCreated event.

**Event broker — transport**

Makes the event available to interested consumers.

**Inventory service — consumer**

Receives the event and begins the stock reservation workflow.

**Notification service — consumer**

Receives the event and sends an order confirmation.

**Data platform — consumer**

Receives the event for analytical processing.

### 11.4 Architecture diagram

```
                   [Order Service]
                         |
                    OrderCreated
                         |
                         v
                   [Event Broker]
                    /     |      \
                   /      |       \
                  v       v        v
          [Inventory] [Notification] [Data Platform]
             Service     Service       Consumer
```

Each consumer performs its own responsibility.

### 11.5 What does loosely coupled mean?

In a tightly coupled design, the order service may need direct knowledge of every downstream service and call each one.

In an event-driven design, the order service publishes an event. Consumers subscribe independently.

This can reduce direct dependencies.

However, it does not mean there are no dependencies. Consumers still depend on event meaning, schema, identifiers and delivery behavior.

### 11.6 Benefits

- New consumers can subscribe without changing the producer's core workflow.
- Different systems can process the same business event.
- Producers and consumers can be developed more independently.
- Event streams can support operational reactions and analytics.
- Systems can absorb temporary differences in processing speed, depending on broker capacity and design.

### 11.7 Challenges

- A consumer may receive an event more than once.
- Events may arrive later than expected.
- Events may be processed out of order.
- A consumer may fail after receiving an event.
- Event schemas may change.
- Tracing a business action across multiple services can be difficult.
- Teams must monitor event lag and failed processing.

### 11.8 Event vs command

An event says that something happened.

Example: OrderCreated.

A command asks a system to do something.

Example: ReserveInventory.

The distinction matters because an event describes a fact, while a command expresses an instruction. Some architectures use both.

### 11.9 Event payload design

A useful event often contains:

- Event identifier.
- Event type.
- Entity or business identifier.
- Event timestamp.
- Schema/version information where needed.
- Relevant business attributes.
- Correlation or trace identifier.

Do not put unnecessary sensitive information into every event. Include only the data consumers need and follow access policies.

---

## 12. Delivery guarantees — why duplicates happen

### 12.1 Why is delivery not always simple?

Imagine a consumer receives a PaymentSucceeded event.

It updates a downstream table. Then the consumer crashes before confirming that it has completed the event.

The broker or consumer may not know whether the business update happened. When processing resumes, the event may be delivered again.

This is one reason duplicate handling matters.

### 12.2 At-most-once delivery

At-most-once means an event is delivered zero or one time.

- It will not be delivered more than once under the stated guarantee.
- But it may be lost if a failure happens at the wrong time.

This may be acceptable for some low-value metrics, but not for every business event.

### 12.3 At-least-once delivery

At-least-once means an event is delivered one or more times.

- The system is designed to avoid losing an event within its guarantee boundary.
- A duplicate may occur.
- The consumer must be able to handle repeated delivery safely.

This is common in distributed systems.

### 12.4 Exactly-once processing

"Exactly once" is often used too casually.

A platform may provide transactional guarantees within a particular boundary. But when a workflow touches multiple independent systems, end-to-end exactly-once business effects require additional design.

For example, writing to a database and sending an external email are two separate side effects. A platform transaction may not make both atomic.

### 12.5 What should a Data Engineer do?

Design for safe repeated processing:

- Use a stable event_id or business key.
- Record which events have been processed.
- Use transactions where appropriate.
- Make writes idempotent.
- Reconcile counts and totals.
- Monitor duplicate rates.

### 12.6 What is idempotency?

An operation is idempotent when repeating it with the same input does not create an unintended additional effect.

Example:

If PaymentSucceeded event EVT900 is received twice, the downstream payment table should still represent one successful payment, not two separate payments.

This can be achieved using unique identifiers and safe database operations.

### 12.7 A simple illustration

First delivery:

| event_id | payment_id | status |
|---|---|---|
| EVT900 | PAY900 | SUCCESS |

Second delivery of the same event:

The consumer checks event_id EVT900 and recognizes that it has already applied the event.

Expected business result: one successful payment record, not two.

The exact implementation depends on database constraints, transaction design and processing framework.

---

## 13. Event ordering and late-arriving events

### 13.1 Why can events arrive out of order?

Suppose an order has two events:

1. OrderCreated at 10:00.
2. OrderShipped at 12:00.

Due to network delays, retries or parallel processing, the consumer may receive OrderShipped before OrderCreated.

If the consumer blindly applies events in arrival order, it may produce an incorrect state.

### 13.2 Event time vs processing time

**Event time:** When the event actually happened in the source system.

**Processing time:** When the processing system handled the event.

Example:

| Event | Time |
|---|---|
| Event occurred | 10:00:00 |
| Event reached the processor | 10:00:08 |
| Processor handled it | 10:00:10 |

Event time is 10:00:00. Processing time is 10:00:10.

The difference may matter for time-based analysis.

### 13.3 What is a late event?

An event is late when it arrives after the expected time window or after a result has already been calculated.

Example:

A delivery update for yesterday arrives today because the delivery partner's system was offline.

The platform needs a policy for correcting the affected result.

### 13.4 How can ordering be managed?

Possible techniques include:

- Partition related events by a stable business key.
- Include event timestamps and sequence/version numbers.
- Reject or quarantine impossible state transitions.
- Buffer events for a defined waiting period.
- Reconcile current state with the source system.
- Use event-time windows and late-event policies.

No single technique solves every ordering problem.

### 13.5 Why is a business key important?

If all events for one order use order_id as the partition key, they may be routed to the same partition in a Kafka-style design. This can preserve their order within that partition.

But it does not automatically guarantee correct ordering across every system or fix events that were published in the wrong order.

---

## 14. How to choose the right processing pattern

Do not choose a technology first. Understand the requirement first.

### 14.1 Ask these questions

**Question 1: How fresh must the result be?**

- Seconds?
- Minutes?
- Hours?
- Next day?

**Question 2: What happens if the result is late?**

- A customer cannot continue checkout?
- An operator sees an old stock level?
- A monthly report is delayed?
- A fraud alert is missed?

**Question 3: How does the data arrive?**

- A database table?
- A file?
- An API?
- A continuous event stream?

**Question 4: How much data arrives?**

- Hundreds of rows?
- Millions of rows?
- Thousands of events per second?

**Question 5: Can records be changed later?**

- Can an order be cancelled?
- Can a payment be refunded?
- Can a shipment status be corrected?

**Question 6: What recovery behavior is needed?**

- Can a batch be rerun?
- Must an event be replayed?
- Can duplicates be tolerated?
- Is a correction report acceptable?

**Question 7: What can the team operate?**

- Does the team have streaming expertise?
- Can it monitor a continuously running service?
- Is the operational cost justified?

### 14.2 Decision table

| Business requirement | Possible starting choice | Reason |
|---|---|---|
| Monthly invoice report | Batch | Monthly result is sufficient |
| Daily sales summary | Batch | Scheduled output meets need |
| Stock dashboard refreshed every 5 minutes | Near-real-time or micro-batch | Small delay is acceptable |
| Payment risk alert within seconds | Streaming / low-latency event processing | Fast reaction has business value |
| Website click collection | Streaming ingestion | Events arrive continuously |
| Historical data migration | Batch | Process defined historical ranges |

These are starting recommendations. Validate latency, volume, recovery and correctness before finalizing architecture.

---

## 15. Practical comparison: five TechMart requirements

Let's examine five requirements in more detail.

### Requirement A — Daily revenue report

**Business request:** Management wants yesterday's revenue by 7 AM.

**Questions to ask:**

- What is the official definition of revenue?
- Are refunds included?
- What is the business time zone?
- When do payment records become available?
- Can late payments update yesterday's report?

**Possible design:** Scheduled batch.

**Why:** A result by 7 AM is sufficient; every transaction does not need to update the report immediately.

**Important checks:** Input counts, payment status, refunds, date boundaries and report completion.

### Requirement B — Payment fraud signal

**Business request:** Identify suspicious payment activity within seconds.

**Questions to ask:**

- What is the allowed decision delay?
- What information is required?
- What happens if the risk service is unavailable?
- Can an event be evaluated more than once?
- How are alerts investigated?

**Possible design:** Low-latency processing, often event-driven.

**Why:** The business needs a rapid reaction to a payment event.

**Important checks:** Duplicate handling, event freshness, service availability and false-positive monitoring.

### Requirement C — Stock dashboard refreshes every five minutes

**Business request:** Warehouse staff want recent stock levels.

**Possible design:** Near-real-time ingestion or a five-minute micro-batch.

**Why:** The requirement allows a small delay. A continuously running complex system may not be necessary.

**Important checks:** Stock update frequency, late events and consistency across warehouses.

### Requirement D — Monthly invoice generation

**Business request:** Generate invoices after the monthly billing period closes.

**Possible design:** Batch.

**Why:** The calculation is period-based and does not need second-by-second output.

**Important checks:** Billing period, duplicate invoices, corrections, rounding rules and audit trail.

### Requirement E — Website click tracking

**Business request:** Capture click events as customers use the website.

**Possible design:** Streaming ingestion, with batch aggregation later.

**Why:** Clicks are generated continuously. The company may want recent activity and historical analysis.

**Important checks:** Event volume, event schema, privacy, duplicate events and retention.

---

## 16. Mini-project: Data Processing Comparison Lab

<badge color="success">Today's practical deliverable</badge>

### 16.1 Project objective

You are a junior Data Engineer at TechMart.

The business has given you five requirements. Your task is to select a suitable processing approach and explain the trade-offs.

You are designing the approach today; you are not expected to deploy Kafka or a production streaming platform.

### 16.2 Project input

| ID | Business requirement | Required freshness |
|---|---|---|
| R1 | Daily revenue report | By 7 AM next day |
| R2 | Payment fraud alert | Within seconds |
| R3 | Stock dashboard | Refresh every 5 minutes |
| R4 | Monthly invoice generation | After month end |
| R5 | Website click collection | Continuous capture |

### 16.3 Step 1 — Understand each business requirement

For every requirement, write:

1. What business problem is being solved?
2. Who uses the output?
3. How fresh must the data be?
4. What happens if the data is late?
5. What source produces the data?

Do not select a technology until these questions are answered.

### 16.4 Step 2 — Select a processing approach

A reasonable initial comparison is:

| ID | Initial choice | Explanation |
|---|---|---|
| R1 | Batch | A daily result by 7 AM is sufficient |
| R2 | Streaming / low-latency | A quick reaction is required |
| R3 | Near-real-time / micro-batch | A five-minute delay is acceptable |
| R4 | Batch | The work is tied to a monthly period |
| R5 | Streaming ingestion | Events arrive continuously |

Your answer may differ if you explain a valid business constraint.

### 16.5 Step 3 — Draw the batch architecture

```
Order Database + Payment Data
             |
             v
      Scheduled Extract
             |
             v
       Batch Validation
             |
             v
     Transform and Aggregate
             |
             v
       Warehouse Tables
             |
             v
       Daily Sales Report
```

Explain each component:

- Scheduled extract: obtains records for the reporting period.
- Batch validation: checks the incoming records.
- Transform and aggregate: applies business rules and calculates totals.
- Warehouse tables: stores the prepared result.
- Daily report: presents the result to management.

### 16.6 Step 4 — Draw the streaming architecture

```
Payment Application
        |
        v
  Payment Event
        |
        v
    Event Broker
        |
        v
 Fraud Risk Consumer
        |
        v
 Risk Result / Alert
```

Explain each component:

- Payment application: creates the event.
- Payment event: describes the payment attempt.
- Event broker: transports and retains events according to its configuration.
- Fraud risk consumer: reads and evaluates events.
- Risk result/alert: makes the outcome available to the appropriate system or team.

### 16.7 Step 5 — Add failure scenarios

| Failure | What could happen? | What should the design consider? |
|---|---|---|
| Daily source extract fails | Report is incomplete | Detect failure, alert and rerun safely |
| Same payment event arrives twice | Duplicate risk evaluation | Stable event ID and idempotent consumer |
| Stock update arrives late | Dashboard is stale | Freshness metric and late-event policy |
| Invoice batch runs twice | Duplicate invoices | Unique invoice key and safe rerun |
| Click consumer stops | Event lag increases | Monitor lag and resume from tracked progress |

### 16.8 Step 6 — Define success measures

Examples:

| Pipeline | Success measure |
|---|---|
| Daily revenue | Report ready by 7 AM; totals reconciled |
| Fraud alert | Defined latency target met; events not silently lost |
| Stock dashboard | Data freshness within five minutes |
| Monthly invoice | One correct invoice per billing key |
| Click ingestion | Event volume and consumer lag monitored |

Success should be measurable. "The pipeline works well" is not a useful operational definition.

### 16.9 Step 7 — Final project deliverable

Submit a short design document containing:

1. Business requirement.
2. Data source.
3. Required freshness.
4. Selected processing pattern.
5. Architecture diagram.
6. Main processing steps.
7. Data quality checks.
8. Failure and recovery plan.
9. Monitoring measures.
10. Reason for choosing the pattern.

### 16.10 Example project explanation

"I compared five TechMart data requirements using business freshness needs. I selected batch processing for the daily revenue report and monthly invoices because scheduled results meet those requirements. I selected low-latency event processing for payment risk because the business needs a fast reaction. For the five-minute stock dashboard, I considered near-real-time or micro-batch processing. I also considered duplicate events, late data, reruns, monitoring and measurable success criteria."

---

## 17. Common beginner mistakes

| Mistake | Why it is incorrect | Better understanding |
|---|---|---|
| OLTP means any database | OLTP describes an operational transaction workload | Focus on the workload |
| OLAP means only Power BI | OLAP is an analytical processing approach | Dashboards are one consumer |
| Streaming is always better | It can add cost and operational complexity | Select based on freshness need |
| Batch means slow | Batch can process large volumes efficiently | It trades immediate freshness for grouped processing |
| Real-time means instant | Every system has measurable latency | Define a target and measurement |
| An event is the same as a command | An event states what happened; a command requests an action | Keep their meanings clear |
| Events always arrive in order | Distributed processing can reorder delivery | Design ordering rules |
| At-least-once means no duplicates | It can deliver the same event repeatedly | Make consumers idempotent |
| Exactly-once is guaranteed everywhere | Guarantees have boundaries | Understand the full end-to-end workflow |
| A successful job proves correct data | Technical completion does not prove business correctness | Validate and reconcile results |
| Every dashboard needs streaming | Some dashboards only need scheduled refresh | Ask users about actual freshness |

---

## 18. Day 2 glossary

| Term | Simple explanation |
|---|---|
| Data processing | Operations that make data useful |
| OLTP | Processing day-to-day business transactions |
| OLAP | Processing analytical queries |
| Transaction | A logical unit of business work |
| Batch | A defined group of records processed together |
| Batch window | The period covered by a batch |
| Batch ID | Identifier for a processing run |
| Backfill | Processing data for an earlier period |
| Stream | A continuous sequence of events |
| Stream processing | Processing events as they arrive |
| Event | A record that something happened |
| Producer | System that publishes an event |
| Consumer | System that reads an event |
| Broker | Infrastructure that transports or retains events |
| Topic | Named event stream in Kafka-style systems |
| Partition | Ordered portion of a topic |
| Offset | Position of a record in a partition |
| Consumer group | Consumers cooperating to read a topic |
| Latency | Time between input/event and result |
| Real-time | Meets a defined low-latency or deadline requirement |
| Near-real-time | Makes data available after a small acceptable delay |
| Event time | Time the event actually happened |
| Processing time | Time the system processed the event |
| At-most-once | Zero or one delivery |
| At-least-once | One or more deliveries |
| Idempotency | Repeating an operation does not create an unintended extra effect |
| Event ordering | The sequence in which events are handled |
| Late event | Event received after the expected time or result window |
| Micro-batch | Small groups processed at short intervals |
| Event-driven architecture | Systems communicate through published events |

---

## 19. Day 2 revision notes

1. Data processing includes reading, validating, filtering, transforming, joining, aggregating, storing and delivering data.
2. OLTP supports day-to-day business transactions.
3. OLAP supports analytical questions across larger or historical datasets.
4. OLTP and OLAP describe workloads, not specific products.
5. Batch processing handles a defined group of records.
6. Streaming processing handles a continuous flow of events.
7. Real-time and near-real-time must be defined using measurable freshness or latency targets.
8. Event-driven architecture uses producers, events, brokers and consumers.
9. Event delivery may involve duplicates, delays and ordering challenges.
10. Safe processing requires identifiers, idempotency, validation and recovery planning.
11. Batch and streaming can coexist in one data platform.
12. Choose the simplest processing approach that meets the business requirement.

---

## 20. Self-assessment

Answer without looking at the notes.

1. What is data processing?
2. What does OLTP stand for?
3. What does OLAP stand for?
4. Give two examples of OLTP operations.
5. Give two examples of OLAP questions.
6. Why can a large analytical query affect a customer-facing application?
7. What is batch processing?
8. What is a batch boundary?
9. What is a backfill?
10. What is stream processing?
11. What is an event?
12. Explain producer, broker and consumer.
13. What is a Kafka topic?
14. What is a partition?
15. What is an offset?
16. What is latency?
17. Explain real-time vs near-real-time.
18. What is at-least-once delivery?
19. Why must consumers handle duplicates?
20. What is idempotency?
21. What is the difference between event time and processing time?
22. Can a platform use both batch and streaming? Give an example.
23. How would you choose a processing pattern for a five-minute stock dashboard?
24. Why should "real-time" be converted into a measurable target?

### Practical assessment

Choose one of the five TechMart requirements. Draw its data flow and explain:

- Why the selected pattern is appropriate.
- What the expected freshness is.
- What can fail.
- How duplicates or late data are handled.
- How the team knows the process is healthy.

**You are ready to move on when:** you can explain OLTP, OLAP, batch, streaming and event-driven architecture in your own words and justify a processing choice using business requirements.

---

## 21. Preview of Day 3

On Day 3, we will study **Data Sources and Formats**.

You will learn:

- Structured, semi-structured and unstructured data.
- Databases, APIs, files and logs as sources.
- CSV, JSON, XML and Parquet.
- How to identify a source's structure and access method.
- How to document sources in the Data Source Inventory mini-project.

Day 2 taught you how data is processed. Day 3 will help you understand the different forms and places from which that data arrives.
