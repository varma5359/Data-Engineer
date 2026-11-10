# Day 2 Interview Preparation: Data Processing Fundamentals

**Mini-project:** Data Processing Comparison Lab

## 1. Core questions

### 1. What is OLTP?
OLTP supports operational transactions such as placing orders, updating inventory and recording payments. It is optimized for many concurrent, short transactions.

### 2. What is OLAP?
OLAP supports analytical queries over integrated or historical data. It is commonly used for aggregations, trends and business reporting.

### 3. Why separate OLTP and OLAP workloads?
Large analytical queries can consume resources needed by customer-facing transactions. Separating workloads helps protect application performance and allows analytical storage to be organized for reporting.

### 4. What is batch processing?
Batch processing collects a bounded set of records and processes them together, often on a schedule. Daily sales aggregation is a typical example.

### 5. What is stream processing?
Stream processing handles a continuous flow of events, allowing systems to react with low latency. Payment events and sensor readings are common examples.

### 6. What is event-driven architecture?
A design in which a producer publishes an event and one or more consumers react to it. Producers and consumers can evolve independently, but duplicate delivery, ordering and schema compatibility must be managed.

## 2. Difference-based questions

| Compare | Answer |
|---|---|
| OLTP vs OLAP | Operational transactions vs analytical queries |
| Batch vs streaming | Bounded group processing vs continuous event processing |
| Real-time vs near-real-time | Strict or very low response target vs small acceptable delay |
| Event time vs processing time | When event occurred vs when processor handled it |
| Producer vs consumer | Publishes events vs reads and processes events |
| At-most-once vs at-least-once | Possible loss without duplicates vs possible duplicates with reduced loss |

## 3. Practical questions

### 7. How would you choose batch or streaming?
I would confirm the business latency target, data volume, event behavior, ordering requirements, recovery needs, team capability and operating cost. I would use batch when delayed results are acceptable and streaming when low latency has clear business value.

### 8. How would you prevent a streaming consumer from applying the same payment twice?
Use a stable event or transaction identifier, record processed identifiers or use a target operation that is idempotent. The exact method depends on the storage system and transaction boundary.

### 9. How would you handle late-arriving events?
Define an event-time policy, allow an appropriate lateness window where needed, update affected aggregates or partitions, and make corrections traceable. The policy depends on the business tolerance for revised results.

## 4. Scenario questions

### Scenario A: A dashboard must refresh every five minutes
**Problem:** Management asks for “real-time” inventory visibility.

**Action:** Clarify whether five-minute freshness is sufficient. Consider incremental batch or micro-batch ingestion, measure end-to-end freshness and alert when the delay exceeds the target.

**Reasoning:** A continuously running streaming platform may be unnecessary if five-minute latency satisfies the business need.

### Scenario B: Payment events are duplicated
**Problem:** A retry causes the same payment event to reach a consumer twice.

**Action:** Use a stable payment/event ID and idempotent processing. Reconcile the resulting payment state with the authoritative payment system.

**Reasoning:** At-least-once delivery can produce duplicates; the business effect must still occur only once.

### Scenario C: Events arrive out of order
**Problem:** `OrderShipped` arrives before `OrderPaid`.

**Action:** Use event timestamps and entity keys, define valid state transitions, buffer or reconcile where appropriate, and avoid blindly overwriting newer state with older events.

**Reasoning:** Arrival order is not always business-event order.

## 5. Troubleshooting

### 10. Batch completed but data is missing
Check input discovery, source query filters, file completeness, rejected records, transformation conditions, load status and reconciliation metrics.

### 11. Streaming lag is increasing
Check consumer throughput, partition assignment, slow downstream writes, retry loops, broker health, message size and recent deployments. Compare incoming rate with processing rate.

### 12. OLTP queries became slow after a report was added
Check query plans and resource utilization. Move heavy analytical work to a reporting or warehouse environment where appropriate, and review indexing and workload scheduling.

## 6. Follow-up questions

- What is a micro-batch?
- What is a dead-letter queue?
- Why are event schemas versioned?
- What is a consumer offset?
- Can streaming systems lose data?
- What does idempotency mean?
- How would you replay an event safely?
- How do you measure data freshness?

## 7. Strong revision points

- Explain workload purpose before naming a technology.
- State the latency requirement explicitly.
- Streaming does not remove the need for validation and recovery.
- Duplicate-safe processing is essential for retryable systems.
- Event time and processing time are different.
- Measure end-to-end freshness, not only job execution time.

## 8. Mini-project defence

For each of the five use cases in the Data Processing Comparison Lab, defend:

1. The chosen processing pattern.
2. The required latency.
3. Expected volume and growth.
4. Duplicate and late-event handling.
5. Failure recovery.
6. Cost and operational trade-offs.

A strong answer explains why a simpler alternative would or would not meet the requirement.
