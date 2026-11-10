# Day 2 Interview Preparation: Data Processing Fundamentals

**Phase:** 1 — Data Engineering Foundations  
**Mini-project:** Data Processing Comparison Lab  
**Target level:** Fresher / Junior Data Engineer

> **Practice method:** Answer in three parts: definition, business example and reason it matters. For scenario questions, explain your assumptions before selecting a technology.

---

## 1. Core concept questions

### 1. What is data processing?

Data processing is the set of operations performed on data to make it useful for a task. It may include reading, filtering, validating, transforming, joining, aggregating, storing and delivering data.

**Example:** Read order and payment records, validate them, combine them and prepare a sales summary.

### 2. What is OLTP?

OLTP stands for Online Transaction Processing. It supports day-to-day business operations such as placing orders, recording payments, updating addresses and reserving inventory.

### 3. What is OLAP?

OLAP stands for Online Analytical Processing. It supports analytical queries such as calculating monthly revenue, comparing regions and studying historical trends.

### 4. What is a transaction?

A transaction is a logical unit of work. For example, placing an order may involve creating an order, recording its items and updating related business state. The system must manage the operation so that partial or conflicting changes do not create incorrect results.

### 5. What is the difference between OLTP and OLAP?

OLTP supports operational transactions, often many short reads and writes. OLAP supports analytical queries, often involving large reads, joins and aggregations over historical or integrated data.

### 6. Is OLTP the same as a database and OLAP the same as a data warehouse?

No. OLTP and OLAP describe workload purposes and processing patterns. A database or warehouse is a system that may support a particular workload.

### 7. Why do companies separate OLTP and OLAP workloads?

Large analytical queries can compete with customer-facing transactions for CPU, memory, storage and connections. A separate analytical environment can protect operational performance and organize data for reporting. Separation is a design decision, not a requirement for every small system.

### 8. What is batch processing?

Batch processing collects a defined group of records and processes them together, often on a schedule. A daily sales report is a common example.

### 9. What is a batch boundary?

A batch boundary defines which records belong to a processing run. It may be a date, time window, file, sequence range or data partition.

### 10. What is a batch ID?

A batch ID identifies one processing run. It helps engineers track input, output, errors and recovery actions for that run.

### 11. What is a backfill?

A backfill is processing data for an earlier period. For example, if Monday's pipeline failed, the team may run a backfill for Monday after fixing the issue.

### 12. What is stream processing?

Stream processing handles a continuous flow of events or small increments as they arrive. It is useful when the business needs low-latency results.

### 13. What is an event?

An event is a record that something happened, such as OrderCreated, PaymentSucceeded or ShipmentDispatched.

### 14. What is event-driven architecture?

It is a design where systems publish events and other systems consume them to react. Producers and consumers can be developed more independently, but delivery, ordering, schema and failure handling must be considered.

### 15. What is latency?

Latency is the time between an input or event becoming available and the result being delivered. It should be measured using a defined start and end point.

---

## 2. Batch, streaming and freshness questions

### 16. What is the difference between batch and streaming?

Batch processes a bounded group of records together. Streaming processes an ongoing sequence of events as they arrive. Batch often suits scheduled reports; streaming can suit low-latency reactions.

### 17. Is batch processing always slower?

No. Batch processing can process large volumes efficiently. Its output may be delayed because it waits for a schedule or a complete group, but the processing itself can be fast.

### 18. Is streaming always better than batch?

No. Streaming may add infrastructure, monitoring, recovery and operational complexity. Use it when lower latency has enough business value to justify those costs.

### 19. What is real-time processing?

Real-time processing aims to meet a defined time constraint. A good requirement states a measurable target, such as a decision returned within 300 milliseconds for a defined percentage of requests.

### 20. What is near-real-time processing?

Near-real-time processing makes data available after a small, acceptable delay, such as a dashboard refresh every five minutes.

### 21. What is the difference between real-time and near-real-time?

Real-time usually has a stricter response-time requirement. Near-real-time allows a small delay. The exact meaning should be agreed and measured for the application.

### 22. What is micro-batch processing?

Micro-batch processing handles small groups of records at short intervals. It can provide frequent updates without maintaining a fully event-by-event processing model.

### 23. Can batch and streaming be used together?

Yes. A platform may ingest events continuously for quick reactions and run batch jobs to create trusted daily summaries for historical reporting.

### 24. How do you decide whether to use batch or streaming?

I clarify the freshness target, consequences of delay, data volume, source behavior, ordering needs, duplicate handling, recovery requirements, team capability and cost. I choose the simplest approach that meets the business need.

---

## 3. Event architecture questions

### 25. What is a producer?

A producer is a system that creates and publishes an event. For example, an order service publishes OrderCreated after an order is successfully created.

### 26. What is a consumer?

A consumer is a system that reads and processes events. An inventory service may consume OrderCreated to begin a stock reservation.

### 27. What is a message or event broker?

A broker is infrastructure that receives, transports or retains messages/events and makes them available to consumers. It is one component of an event-driven architecture.

### 28. What is Apache Kafka?

Apache Kafka is a distributed event-streaming platform used to publish, store and consume streams of records. Important concepts include topics, partitions, producers, consumers, consumer groups and offsets.

### 29. What is a Kafka topic?

A topic is a named stream of related records. Examples include orders, payments and shipments.

### 30. What is a partition?

A partition is an ordered portion of a topic's log. Partitions support distribution and parallel processing. Ordering is generally maintained within a partition, not automatically across all partitions.

### 31. What is an offset?

An offset identifies a record's position within a partition. Consumers use offsets to track progress.

### 32. What is a consumer group?

A consumer group is a set of consumers cooperating to read a topic. Partitions are assigned among group members to distribute work.

### 33. What is the difference between an event and a command?

An event describes something that happened, such as OrderCreated. A command requests an action, such as ReserveInventory. An event is a fact; a command is an instruction.

### 34. What does loosely coupled mean in event-driven architecture?

The producer does not need to directly call and coordinate every consumer. It publishes an event, and interested consumers subscribe independently. They still depend on a shared event contract and its meaning.

---

## 4. Delivery and correctness questions

### 35. What is at-most-once delivery?

At-most-once means an event is delivered zero or one time. It avoids duplicate delivery within the guarantee but may allow an event to be lost.

### 36. What is at-least-once delivery?

At-least-once means an event is delivered one or more times. It reduces the risk of loss within the guarantee, but duplicate delivery is possible.

### 37. What is exactly-once processing?

Exactly-once guarantees are bounded by the platform and workflow. A system may provide transactional guarantees inside a defined boundary, but end-to-end business effects across independent systems need additional design.

### 38. Why do duplicate events happen?

A consumer may process an event but fail before acknowledging or committing progress. When it restarts, the event may be delivered again. Retries and network failures can also lead to repeated delivery.

### 39. What is idempotency?

An operation is idempotent when repeating it with the same input does not create an unintended additional effect. It is useful for safe retries and duplicate event handling.

### 40. How can you prevent a payment event from being applied twice?

Use a stable event or payment identifier, enforce uniqueness where appropriate, record processed events and make the database update idempotent. The operation should be designed with suitable transaction boundaries.

### 41. What is event ordering?

Event ordering is the sequence in which events are produced or processed. Distributed systems may deliver or process events out of order, especially when retries and parallel processing are involved.

### 42. How can ordering be supported in Kafka-style processing?

Use a suitable partition key so related events are routed to the same partition. Ordering is maintained within that partition, subject to producer and consumer design. This does not automatically guarantee end-to-end business ordering.

### 43. What is event time?

Event time is when the event actually occurred in the source system.

### 44. What is processing time?

Processing time is when the processing application handled the event. It may be later than event time because of network or queue delays.

### 45. What is a late-arriving event?

A late-arriving event reaches the processing system after the expected time window or after a result has already been calculated. The pipeline needs a policy to correct or update affected outputs.

---

## 5. Scenario-based questions

### 46. A manager wants a daily sales report by 7 AM. Would you use streaming?

Not automatically. I would confirm when source data becomes available and how fresh the report must be. If a scheduled result by 7 AM is sufficient, batch processing is likely simpler and appropriate.

### 47. A payment-risk service must react within seconds. What would you consider?

I would consider a low-latency event-driven design. I would define the latency target, event schema, duplicate handling, failure behavior, monitoring, security and what happens if the risk service is unavailable.

### 48. A stock dashboard needs updates every five minutes. What would you choose?

Near-real-time ingestion or a five-minute micro-batch may be sufficient. I would compare the actual update requirement, source change frequency, cost and recovery behavior before choosing.

### 49. The same payment event is received twice. What should happen?

The consumer should identify the repeated event using a stable identifier and avoid applying the same business effect twice. It should also record enough information for monitoring and investigation.

### 50. The consumer crashes after writing to the database but before committing its offset. What could happen?

The event may be delivered again after restart. If the write is not idempotent, the business effect may be duplicated. A stable key, uniqueness rule and transaction design can make the retry safe.

### 51. A daily batch loaded half its records and then failed. What would you do?

I would inspect the batch status, input/output counts and failure point. I would determine which records were committed and whether the load can be safely rerun. I would use staging, transactions, batch identifiers or merge logic as appropriate, then reconcile the final result.

### 52. A source file arrives two hours late. What would you check?

I would check the reporting deadline, file date, source delivery history and whether the report was already published. Then I would follow the late-data policy, such as a correction run or reprocessing the affected date.

### 53. A dashboard says "real-time" but refreshes every 15 minutes. Is that necessarily wrong?

Not necessarily, but the label may be misleading. I would define the business freshness target and describe the actual 15-minute delay clearly.

### 54. An event arrives before the event that created its parent record. What could go wrong?

The consumer may not find the expected parent record or may apply an invalid state transition. The design may need buffering, retry, ordering by key, version checks or reconciliation.

### 55. The business asks to make every report streaming. What would you say?

I would ask what decision needs to happen faster and what delay is acceptable. Reports that only need daily or hourly freshness may be better served by batch. Streaming should be selected when the business benefit justifies its complexity and cost.

---

## 6. Mini-project interview: Data Processing Comparison Lab

### 56. Explain your Day 2 mini-project.

"I compared five TechMart requirements and selected processing approaches based on freshness and business impact. I chose batch for daily revenue and monthly invoices, low-latency event processing for payment-risk alerts, near-real-time or micro-batch for the five-minute stock dashboard, and streaming ingestion for continuous website click capture. I documented failure handling, duplicate protection and monitoring."

### 57. Why did you choose batch for daily revenue?

The report is required by 7 AM, not after every individual order. A scheduled batch can meet the deadline with a simpler operating model.

### 58. Why did you choose event processing for payment-risk alerts?

The business needs to react quickly to payment attempts. Events can be delivered to a risk consumer for low-latency evaluation.

### 59. Why might micro-batch be enough for stock updates?

If the dashboard only needs a refresh every five minutes, processing small groups at short intervals may meet the requirement without the full complexity of continuous event-by-event processing.

### 60. Why did you include failure handling in the project?

A design must explain what happens when sources, consumers or destinations fail. Without recovery and monitoring, the system may silently miss or duplicate data.

### 61. What would you monitor in the batch design?

Run status, duration, input count, output count, rejected count, freshness, reconciliation results and whether the report was delivered on time.

### 62. What would you monitor in the streaming design?

Event throughput, consumer lag, processing errors, duplicate rate, event freshness, broker health and alert delivery.

### 63. What would you improve before production?

I would define exact latency targets, source contracts, event schemas, retention, access controls, replay and recovery procedures, alert ownership, cost limits and service-level objectives. I would test failure and duplicate scenarios before release.

---

## 7. Difference-based quick revision

| Compare | Key distinction |
|---|---|
| OLTP vs OLAP | Operational transactions vs analytical workloads |
| Batch vs streaming | Bounded group processing vs continuous event processing |
| Real-time vs near-real-time | Strict low-latency target vs small acceptable delay |
| Producer vs consumer | Publishes events vs reads events |
| Event vs command | Describes what happened vs requests an action |
| Event time vs processing time | When event happened vs when it was handled |
| At-most-once vs at-least-once | Zero/one delivery vs one-or-more delivery |
| Topic vs partition | Named stream vs ordered segment of a stream |
| Batch ID vs event ID | Identifies a batch run vs identifies an event |
| Backfill vs rerun | Process historical missing data vs execute a run again |

---

## 8. Common weak answers and stronger answers

| Weak answer | Stronger answer |
|---|---|
| "OLTP is a database." | "OLTP is a workload for operational transactions." |
| "OLAP is Power BI." | "OLAP is analytical processing; Power BI is one possible consumer." |
| "Streaming is always faster and better." | "Streaming may reduce latency, but adds operational complexity and should match a real business need." |
| "Batch is old technology." | "Batch is appropriate for scheduled workloads where delayed results are acceptable." |
| "Exactly once means duplicates never happen." | "Exactly-once guarantees have boundaries; end-to-end effects need careful design." |
| "Kafka guarantees every event is in global order." | "Ordering is generally maintained within a partition; key and partition design matter." |
| "We just retry when it fails." | "Retries need idempotency and a recovery policy to avoid duplicate effects." |

---

## 9. Practice tasks

### Task 1 — One-minute explanation

Explain OLTP and OLAP using an online shopping example.

### Task 2 — Pattern selection

Choose batch, near-real-time or streaming for:

1. Monthly billing.
2. Payment-risk alert.
3. Daily sales report.
4. Stock dashboard refreshed every five minutes.
5. Continuous website click capture.

Give one reason for each choice.

### Task 3 — Event flow

Draw a producer, broker and two consumers for an OrderCreated event. Explain what each component does.

### Task 4 — Failure explanation

A consumer receives a payment event twice. Explain how an idempotent design prevents duplicate business effects.

### Task 5 — Project defense

Present the Data Processing Comparison Lab. Explain your assumptions, selected patterns, failure handling and monitoring.

---

## 10. Final interview checklist

- [ ] I can explain OLTP and OLAP without confusing them with products.
- [ ] I can explain batch processing and batch boundaries.
- [ ] I can explain streaming and event-driven architecture.
- [ ] I can define real-time and near-real-time using measurable targets.
- [ ] I understand producers, consumers, brokers, topics, partitions and offsets.
- [ ] I can explain duplicate delivery and idempotency.
- [ ] I can explain event time and processing time.
- [ ] I can justify a processing pattern using a business requirement.
- [ ] I can defend my Data Processing Comparison Lab.

**Interview rule:** Do not recommend Kafka, Spark or another tool just because you know its name. Explain the requirement first, then explain why the tool or processing pattern fits.
