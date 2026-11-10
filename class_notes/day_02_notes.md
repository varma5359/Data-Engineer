# Day 2 — Data Processing Fundamentals
**Pathway:** TechPhile.ai Data Engineer | **Phase:** Foundations | **Level:** Beginner

## Learning objectives
Understand OLTP and OLAP, batch and stream processing, latency terminology, and event-driven architecture.

## 1. OLTP — Online Transaction Processing
OLTP systems support day-to-day business operations. They frequently insert, update, and retrieve individual records.

Examples: placing an order, updating a customer's address, recording a payment.

Typical characteristics:
- Many short transactions.
- Current operational state.
- Strong consistency and transaction handling.
- Tables often normalized to reduce update anomalies.
- Application users are primary consumers.

Example: an orders table in an e-commerce application.

## 2. OLAP — Online Analytical Processing
OLAP systems support analysis across many records: trends, totals, comparisons, and business reporting.

Examples: monthly revenue by region, repeat customer rate, inventory trends.

Typical characteristics:
- Read-heavy analytical queries.
- Large scans and aggregations.
- Historical data.
- Models optimized for reporting (often facts and dimensions).
- Analysts and BI tools are common consumers.

## OLTP vs OLAP
| OLTP | OLAP |
|---|---|
| Operates the business | Analyzes the business |
| Insert/update individual records | Scan, join, aggregate many records |
| Current operational data | Current plus historical data |
| Application workload | Reporting/analytical workload |
| Often normalized | Often dimensional/denormalized |

**Why separate them?** Heavy analytical queries can slow operational applications. A data platform copies and transforms data for analytics without burdening the production application.

## 3. Batch processing
Batch processing collects a bounded set of data and processes it at a scheduled time or when a batch is ready.

Flow: `Collect records → Process batch → Validate → Publish`

Examples:
- Daily sales aggregation.
- Nightly customer snapshot.
- Hourly inventory file ingestion.

Advantages: simpler operations, efficient bulk processing, easier repeatability for many workloads.
Trade-off: results are delayed until the next batch.

## 4. Stream processing
Stream processing handles events continuously or in small windows as they arrive.

Examples:
- Payment fraud signals.
- Live clickstream activity.
- Device telemetry.
- Near-real-time inventory updates.

A stream is not automatically “one event at a time forever”; systems may use windows, buffering, checkpoints, and event-time handling.

## Batch vs streaming
| Batch | Streaming |
|---|---|
| Bounded dataset/run | Continuous event flow |
| Scheduled or triggered | Event-driven/continuous |
| Minutes to hours may be acceptable | Lower latency often required |
| Easier rerun of a defined batch | Requires state, checkpoints, late-event strategy |
| Example: daily revenue | Example: live fraud alert |

Choose based on business latency and operational requirements—not fashion.

## 5. Real-time, near-real-time, and batch latency
- **Real-time:** response within a strict, defined time bound (the bound depends on the business/system).
- **Near-real-time:** small delay, often seconds or minutes, acceptable to the business.
- **Batch:** data is processed at intervals; delay may be hours or a day.

Ask the business: “How fresh must the result be?” A 5-minute delay may be real-time enough for a dashboard but not for a safety control.

## 6. Event-driven architecture
An event records that something happened: `OrderPlaced`, `PaymentCompleted`, `InventoryChanged`.

Flow:
`Application emits event → Event broker/stream → Consumer → Store or action`

Benefits:
- Producers and consumers are less tightly coupled.
- Multiple consumers can react to the same event.
- Processing can scale independently.

Engineering concerns: duplicate delivery, ordering, schema compatibility, retries, dead-letter handling, and consumer lag.

## 7. Scenario — retail inventory
- Warehouse stock reconciliation: batch may be appropriate.
- Website availability after purchase: near-real-time updates may be needed.
- Stock movement audit: retain events and reconcile periodically.

A hybrid architecture is common: streaming for rapid updates plus batch jobs for reconciliation and historical analytics.

## 8. Mini-project — Data Processing Comparison Lab
Create a comparison sheet for three requirements:
1. Monthly revenue report.
2. Payment fraud alert.
3. Inventory dashboard updated every 3 minutes.

For each, write: source, processing style, acceptable latency, failure impact, and reason. Draw one batch flow and one event flow.

## 9. Common mistakes
- Calling every scheduled job “streaming.”
- Assuming streaming guarantees exactly-once business effects.
- Ignoring late events and duplicate events.
- Using a streaming platform when hourly batch is sufficient.
- Running analytics directly on a busy transactional database without workload planning.

## Quick revision
- OLTP runs operational transactions; OLAP supports analysis.
- Batch processes bounded data; streaming handles ongoing events.
- Event-driven systems react to events.
- Latency is a business requirement; architecture follows it.
