# Day 2 — Data Processing Fundamentals | Interview Preparation

## Core Q&A

### 1. What is OLTP?
OLTP supports operational transactions such as order creation and payment updates. It handles many short reads/writes and prioritizes transaction correctness.

### 2. What is OLAP?
OLAP supports analytical workloads such as trend analysis and large aggregations over historical data. It is optimized for reading and exploring datasets.

### 3. Why should analytical workloads be separated from OLTP?
Analytical scans and joins can consume resources and affect application response time. Replicating data into an analytical platform isolates workloads and enables historical modeling.

### 4. What is batch processing?
Processing a defined set of records together, usually on a schedule or trigger. Example: load yesterday's transactions and calculate daily revenue.

### 5. What is stream processing?
Processing a continuous flow of events, often with low latency. Example: react to payment events to detect suspicious activity.

### 6. Batch vs streaming?
Batch is bounded and periodic; streaming is continuous/event-driven. Streaming has additional operational concerns such as state, event ordering, late arrivals, checkpoints, and consumer lag.

### 7. Real-time vs near-real-time?
Both require a latency target. Real-time usually has a strict response bound; near-real-time accepts a small delay. Clarify the actual SLA rather than relying on labels.

### 8. What is event-driven architecture?
Components communicate through events representing facts that occurred. Producers publish events; consumers process them independently. This reduces coupling but requires careful handling of duplicates, failures, and schemas.

## Comparison table
| Question | OLTP | OLAP |
|---|---|---|
| Main purpose | Run business operations | Analyze business data |
| Typical query | Find/update one order | Revenue by month and region |
| Data | Current operational | Historical and integrated |
| Workload | Transactional | Analytical |

## Scenario: choose processing pattern
- Payroll calculation once per month → batch.
- Fraud signal during payment → stream/low latency.
- Dashboard refreshed every 10 minutes → micro-batch or scheduled batch may suffice.
- Daily source reconciliation → batch, even if operational events are streamed.

## Troubleshooting stream systems
If dashboard lag increases:
1. Check producer rate and consumer rate.
2. Inspect consumer lag and processing duration.
3. Look for downstream database/API bottlenecks.
4. Check partition/key skew.
5. Review errors, retries, and backpressure.
6. Scale or optimize the actual bottleneck; don't blindly add consumers.

## Follow-up prompts
- What is a processing window?
- What is event time vs processing time?
- What is consumer lag?
- How do batch and streaming coexist?
- Why can events be delivered more than once?

## Quick revision
- Explain OLTP/OLAP with an order application example.
- Pick batch vs streaming from latency and failure requirements.
- Mention duplicate events, late arrivals, and recovery for streaming scenarios.
