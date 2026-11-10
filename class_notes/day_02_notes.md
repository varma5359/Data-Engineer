# Day 2: Data Processing Fundamentals

**Phase:** 1 — Data Engineering Foundations  
**Learning time:** 2–3 hours  
**Mini-project:** Data Processing Comparison Lab

## 1. Learning outcomes

- Explain OLTP and OLAP using business examples.
- Distinguish batch, streaming and near-real-time processing.
- Understand event-driven architecture at a beginner level.
- Select a processing pattern based on latency, volume and business need.
- Explain why operational and analytical workloads are often separated.

## 2. Begin with two business questions

An online store has an order application and a management dashboard.

**Application question:** “Can customer 1001 place an order now?”

The system must validate inventory, create the order and record payment reliably. This is an operational transaction.

**Management question:** “How did revenue change across all regions over the last 18 months?”

The system must scan and aggregate a large historical dataset. This is an analytical query.

These workloads have different goals and access patterns.

## 3. OLTP — Online Transaction Processing

OLTP systems support the daily operations of an organization.

Examples:

- Creating an order.
- Updating a customer's address.
- Recording a payment.
- Reserving inventory.
- Updating a delivery status.

Typical characteristics:

- Many short transactions.
- Frequent inserts and updates.
- Current operational data.
- Strong consistency requirements.
- Queries often access a small number of records.
- Concurrent users and applications.

Example order table:

| order_id | customer_id | order_status | order_total |
|---|---|---|---:|
| 501 | 1001 | PAID | 1250 |
| 502 | 1002 | PENDING | 800 |

An application may update order 502 from PENDING to PAID after payment confirmation.

## 4. OLAP — Online Analytical Processing

OLAP systems support analysis across large amounts of historical or integrated data.

Examples:

- Monthly revenue by region.
- Customer retention by acquisition channel.
- Product performance over time.
- Inventory trends across warehouses.

Typical characteristics:

- Complex read-heavy queries.
- Aggregations and joins.
- Historical analysis.
- Data from multiple operational sources.
- Often organized for analytical access.

Example analytical question:

“Show monthly paid revenue by product category for the last year.”

The query may read millions of order lines and aggregate them by month and category.

## 5. OLTP vs OLAP

| Dimension | OLTP | OLAP |
|---|---|---|
| Main purpose | Run the business | Understand the business |
| Typical user | Application/customer/service staff | Analyst/manager/data scientist |
| Workload | Many small transactions | Large analytical queries |
| Data | Current operational records | Historical and integrated data |
| Operations | Insert, update, point lookup | Scan, join, aggregate |
| Design | Transaction integrity and efficient writes | Query efficiency and analytical structure |
| Example | Place an order | Analyze yearly sales |

A production organization may use different systems for these workloads to prevent large analytical queries from slowing down customer transactions.

## 6. Why separate operational and analytical workloads?

Imagine a dashboard runs a large query every minute against the same database that processes customer orders.

Possible impact:

- More CPU and memory usage.
- Slower application queries.
- Lock contention in some workloads.
- Unpredictable response times.
- Risk to customer-facing operations.

A common architecture copies or streams data into an analytical platform. This allows each environment to serve its intended workload.

Separation is a design choice, not a universal rule. Small systems may begin with one database and separate workloads as needs grow.

## 7. Batch processing

**Batch processing** collects a bounded group of records and processes them together.

Examples:

- Import all sales files received today at midnight.
- Calculate yesterday's revenue every morning.
- Generate monthly billing summaries.
- Process a daily inventory snapshot.

Flow:

`Collect records -> Validate batch -> Transform -> Load -> Report status`

Advantages:

- Straightforward to schedule and reason about.
- Efficient for large groups of records.
- Easier to rerun a defined batch when designed well.
- Often simpler and cheaper than continuous processing.

Limitations:

- Results are not available until the next run.
- A failed batch can delay downstream reports.
- Large batches may need partitioning and checkpoints.

### Batch example

A retailer receives a file containing 50,000 orders every night. At 1 AM, the pipeline validates the file, calculates line totals and loads the warehouse.

The management dashboard is updated after the batch completes.

## 8. Streaming processing

**Streaming processing** handles a continuous flow of events as they arrive.

Examples:

- Card payment authorization events.
- Website click events.
- IoT sensor readings.
- Delivery location updates.
- Fraud alerts.

Flow:

`Event producer -> Event transport -> Stream processor -> Destination`

Streaming can reduce the time between an event occurring and the system reacting to it.

Important concepts:

- Event: a record that something happened.
- Producer: a system that creates events.
- Consumer: a system that reads events.
- Broker: infrastructure that transports and retains events.
- Offset or position: a consumer's progress through a stream.
- Event time: when the event actually occurred.
- Processing time: when the system processed it.

Kafka is one commonly used event-streaming platform. It is introduced here conceptually; implementation comes later in the pathway.

## 9. Batch vs streaming

| Dimension | Batch | Streaming |
|---|---|---|
| Input | Bounded group | Ongoing events |
| Processing | Scheduled or triggered | Continuous or micro-batched |
| Latency | Minutes to hours, sometimes longer | Seconds or less, depending on design |
| Complexity | Often lower | Often higher |
| Recovery | Rerun a batch or partition | Replay from a known position/checkpoint |
| Example | Daily sales report | Payment-risk alert |

Do not select streaming just because it sounds modern. Select it when the business value of low latency justifies the operational complexity.

## 10. Real-time and near-real-time

These terms describe how quickly a system makes data available.

- **Real-time:** processing and response occur within a defined, usually strict time constraint.
- **Near-real-time:** data becomes available with a small delay, such as seconds or minutes.
- **Batch:** data is processed at scheduled intervals.

The business must define acceptable latency. “Real-time” without a measurable target is ambiguous.

Examples:

| Use case | Likely pattern |
|---|---|
| Fraud alert during payment | Real-time or low-latency streaming |
| Inventory dashboard refreshed every 5 minutes | Near-real-time |
| Monthly finance report | Batch |
| Daily product recommendations refresh | Batch or scheduled incremental |
| Sensor safety alarm | Real-time streaming |

## 11. Event-driven architecture

In an event-driven design, one system publishes an event when something happens. Other systems react to that event.

Example:

1. Customer places an order.
2. Order service publishes `OrderCreated`.
3. Inventory service receives the event and reserves stock.
4. Notification service sends an order confirmation.
5. Data platform captures the event for analytics.

Benefits:

- Systems can be loosely coupled.
- Multiple consumers can react independently.
- Producers do not need to know every downstream consumer.

Challenges:

- Events may be duplicated.
- Consumers may process events out of order.
- A consumer may be temporarily unavailable.
- Event schemas can change.
- Tracing one business transaction across services requires correlation identifiers.

An event is a statement that something happened; it is not automatically a command to perform an action.

## 12. Delivery and ordering concepts

### At-most-once delivery
An event is delivered zero or one time. Duplicates are avoided, but a failure may result in loss.

### At-least-once delivery
An event is delivered one or more times. Loss is reduced, but consumers must handle duplicates.

### Exactly-once claims
Exactly-once behavior is difficult across multiple independent systems. A platform may provide transactional guarantees within a defined boundary, but end-to-end business correctness still requires careful design.

Practical rule: design consumers to be idempotent wherever possible.

### Ordering
Events may arrive out of order due to retries, parallel processing or network delays. If order matters, define the ordering key and how late events are handled.

## 13. Choosing a processing pattern

Ask these questions:

1. How quickly does the business need the result?
2. How many records or events arrive?
3. Are records updated after creation?
4. Can the consumer tolerate duplicates?
5. Does event order matter?
6. What is the acceptable recovery time?
7. Does the team have the skills to operate streaming infrastructure?
8. What is the cost of delayed data compared with the cost of continuous processing?

Example: a daily finance reconciliation may be safely batched. A payment authorization decision may need a low-latency response.

## 14. Mini-project: Data Processing Comparison Lab

### Scenario

A retail company has five requirements.

| Requirement | Latency need | Candidate pattern |
|---|---|---|
| Daily revenue report | Next morning | Batch |
| Payment fraud signal | Seconds | Streaming |
| Stock dashboard refresh | 5 minutes | Near-real-time |
| Monthly invoice generation | Monthly | Batch |
| Website click tracking | Continuous capture | Streaming ingestion |

### Step 1 — Write down the business requirement

For each use case, identify:

- Source of data.
- Expected volume.
- Acceptable delay.
- What happens if data is late.
- Whether event ordering matters.
- Who consumes the result.

### Step 2 — Draw the architecture

Batch flow:

`Database/files -> Scheduled extraction -> Batch transform -> Warehouse`

Streaming flow:

`Application -> Event broker -> Consumer -> Operational alert / analytical store`

### Step 3 — Identify failure handling

| Failure | Batch response | Streaming response |
|---|---|---|
| Temporary destination outage | Retry the batch safely | Pause/retry consumer |
| Duplicate input | Deduplicate by stable key | Idempotent consumer |
| Invalid record | Quarantine and report | Dead-letter or quarantine path |
| Late record | Reprocess affected partition | Late-event policy |
| Schema change | Validate before load | Version and compatibility checks |

### Step 4 — Explain your decision

For each use case, write one paragraph:

- Why the selected pattern fits.
- What trade-off it introduces.
- How correctness will be measured.
- How recovery will work.

## 15. Common mistakes

| Mistake | Correction |
|---|---|
| Calling every scheduled job real-time | State a measurable latency target |
| Assuming streaming is always better | Compare business value and operating cost |
| Treating OLTP and OLAP as file formats | They describe workload and system purpose |
| Assuming an event is processed once | Design for retries and duplicates |
| Ignoring event time | Distinguish event time from processing time |
| Assuming event order is automatic | Define keys and ordering requirements |
| Treating a broker as the whole data platform | Include consumers, storage, quality and operations |

## 16. Day 2 revision

- OLTP runs operational applications; OLAP supports analytical questions.
- Batch processes bounded groups of records.
- Streaming processes ongoing events.
- Near-real-time describes low but non-zero delay.
- Event-driven systems react to published events.
- Delivery guarantees and ordering affect correctness.
- Choose the simplest pattern that meets the business latency and reliability needs.

## 17. Check your understanding

1. Why can a large analytical query affect an order application?
2. Is a five-minute dashboard refresh necessarily streaming?
3. What is the role of an event broker?
4. Why must consumers handle duplicate events?
5. What is the difference between event time and processing time?
6. Which pattern would you choose for monthly billing and why?
