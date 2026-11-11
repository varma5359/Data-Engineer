# Day 39 Interview Guide — Pipeline Logging and Observability

**Project:** TechStore Pipeline Observability Tool  
**Audience:** Fresher / Junior Data Engineer  
**Interview method:** Explain what the signal means, how it is collected, what threshold or context makes it useful, and what action follows.

---

## Part 1 — Core concepts with complete answers

### 1. What is logging?

**Model answer:** Logging is recording events that occur while a program or pipeline runs. Useful events include pipeline start, source connection, records extracted, validation completed, destination write completed and failure details. Logs help an operator reconstruct what happened. A good log includes context such as run ID, pipeline name, stage and safe error information.

### 2. What is a metric?

**Model answer:** A metric is a numeric measurement collected over time. Examples include rows read, rows rejected, execution duration, retry count, source lag and output freshness. Metrics are useful for trends, thresholds, dashboards and alerts. A metric should have a clear definition and consistent unit.

### 3. What is observability?

**Model answer:** Observability is the ability to understand a system's internal behavior from the information it emits. In a data pipeline, logs explain individual events, metrics show numeric trends and traces show how work moves across components. Together they help answer what happened, where it happened, how much data was affected and why the result may be unhealthy.

### 4. Logs versus metrics versus traces?

**Model answer:**
- Logs are event records with context, such as “batch 4 failed”.
- Metrics are numeric measurements over time, such as rejection rate or processing duration.
- Traces connect operations across components and show where time or failure occurred in a distributed request or workflow.

They complement one another. A metric may reveal that duration increased; logs explain a particular run; a trace may show that the database call consumed most of the time.

### 5. Why does every pipeline run need a run ID?

**Model answer:** A run ID correlates all logs, metrics and output metadata belonging to one execution. If two runs execute concurrently, the run ID helps distinguish their events. It also connects source files, checkpoint updates, rejected records and output manifests to the same execution. A run ID should not contain secrets or personal information.

### 6. What is structured logging?

**Model answer:** Structured logging records named fields, often as JSON, rather than only free-form text. For example, a log event can contain `run_id`, `stage`, `status`, `rows_read` and `duration_seconds`. Search and monitoring systems can filter and aggregate those fields reliably. The field names and types should be consistent.

### 7. What should a data pipeline log?

**Model answer:** At minimum, log pipeline name, run ID, stage, start/end time, status, source/target identifiers, rows read, accepted, rejected and written, duration, retry count, checkpoint boundary and safe error type/message. Log enough to diagnose the run, but avoid secrets and unnecessary personal data.

### 8. What should never be logged?

**Model answer:** Passwords, access tokens, private keys, full connection strings containing credentials, payment card details and sensitive personal data should not be written to ordinary logs. If an identifier is needed for troubleshooting, use an approved masked or pseudonymous form. Logs need access control and retention policies because they can contain operationally sensitive information.

### 9. What is a pipeline status?

**Model answer:** A status is the lifecycle state of a run, such as RUNNING, SUCCEEDED, FAILED or CANCELLED. A pipeline should not be marked SUCCEEDED merely because the script reached its last line. It should pass required validations, reconciliation and publication checks. A zero-row run may be valid or an incident depending on the expected source behavior.

### 10. Why measure rows read, accepted, rejected and written?

**Model answer:** These counts help detect silent data loss and explain pipeline behavior. For a simple one-to-one process, input rows should reconcile with accepted plus rejected rows. Written rows should match accepted rows unless the pipeline intentionally aggregates or filters data. Count metrics make a “successful but empty” or unexpectedly small run visible.

### 11. What is data freshness?

**Model answer:** Freshness describes how up to date the target data is relative to the source or business event time. It can be measured as the difference between the current time and the latest successfully processed event or source update. A pipeline can finish quickly but still produce stale data if the source feed is delayed.

### 12. What is latency?

**Model answer:** Latency is the time taken for data or an operation to move from one point to another. For a data pipeline, it may mean event-to-target delay, source extraction duration or stage execution time. The measurement must specify its start and end points; otherwise two teams may use the same word for different things.

### 13. What is throughput?

**Model answer:** Throughput is the amount of work processed per unit of time, such as rows per second or megabytes per minute. It helps compare processing capacity, but high throughput does not prove correctness. I evaluate throughput together with data-quality, error and freshness metrics.

### 14. What is an SLI?

**Model answer:** A Service Level Indicator is a measured value representing service behavior. Examples include percentage of daily runs completed before the deadline, successful task ratio or data freshness lag. The SLI must have a precise definition and measurement window.

### 15. What is an SLO?

**Model answer:** A Service Level Objective is a target for an SLI over a defined period. For example, 99% of daily order pipelines should publish validated data before 7 AM over a rolling 30-day window. An SLO is an internal reliability target; it should be measurable and tied to business expectations.

### 16. What is an SLA?

**Model answer:** A Service Level Agreement is a formal commitment between a service provider and customer, often including service targets and consequences or remedies. An SLA may be informed by SLOs and SLIs, but the terms are not interchangeable.

### 17. What is alert fatigue?

**Model answer:** Alert fatigue occurs when operators receive too many low-value or repeated alerts and begin ignoring them. I reduce it by alerting on actionable conditions, deduplicating repeated alerts, setting meaningful thresholds, grouping related failures and providing runbook instructions. Alerts should indicate who needs to act and what action is expected.

### 18. What is metric cardinality?

**Model answer:** Cardinality is the number of distinct values for a metric label or dimension. If every customer_id or run_id becomes a metric label, the number of time series can grow dramatically and burden the monitoring system. High-cardinality identifiers belong in logs or traces more often than in metric labels. Metrics should use bounded dimensions such as pipeline name, stage and status.

### 19. Why is a dashboard not the same as observability?

**Model answer:** A dashboard displays selected signals and helps operators see trends or status. Observability is broader: it includes emitting useful signals, correlating them, querying them and investigating unexpected behavior. A dashboard with poor or incomplete metrics cannot make a system observable.

### 20. What is a runbook?

**Model answer:** A runbook is an operational document explaining how to diagnose and respond to a known condition. It should state the alert meaning, checks to perform, safe recovery steps, escalation path and verification required before closing the incident. It should not tell an operator to blindly rerun a pipeline without checking checkpoint and target state.

### 21. How do you monitor data quality?

**Model answer:** I monitor rule-based signals such as null rates, duplicate-key counts, accepted/rejected counts, schema changes, source-to-target count differences and important aggregate totals. I compare them with expected ranges or business thresholds. Technical job success and data quality success are separate conditions.

### 22. How do you monitor pipeline duration?

**Model answer:** I measure start and end timestamps for the whole run and important stages, then calculate durations in a consistent unit. I compare current duration with historical patterns and the business deadline. A threshold should account for normal variation; a sudden large increase may indicate source, network, query or resource problems.

### 23. What is an alert threshold?

**Model answer:** An alert threshold is a condition that triggers an alert, such as freshness lag exceeding 60 minutes or rejection rate exceeding an agreed limit. Thresholds should be based on business impact and historical behavior. Too low creates noise; too high may detect incidents too late. Some metrics need anomaly detection rather than one fixed value.

### 24. Why use a run summary?

**Model answer:** A run summary provides a compact record of the execution: run ID, status, timestamps, stage durations, counts, retry totals, checkpoint, reconciliation result and safe failure information. It helps compare runs, support audits and drive dashboards or alerting.

---

## Part 2 — Project-based questions

### 25. Explain the TechStore Pipeline Observability Tool.

**Model answer:** The project creates a unique run ID and a run context, configures Python logging, records pipeline stage events and captures processing counters. It calculates duration and data-quality metrics, checks reconciliation and freshness conditions, evaluates health rules, and writes a JSON run summary. It then investigates an unhealthy run using the recorded evidence. The project demonstrates how an operator can understand a run instead of relying on a single SUCCESS message.

### 26. Why did you use a run ID?

**Model answer:** The run ID ties together all stage logs and the summary for one execution. If the pipeline runs twice or concurrently, I can filter events for one run and inspect its counts and failures without mixing them with another execution.

### 27. How would you detect a successful run that processed zero rows unexpectedly?

**Model answer:** I compare rows_read with the expected behavior for that source and time window. If zero rows are unusual, I raise a warning or fail the quality gate depending on the contract. I inspect source availability, filters, pagination, date boundaries and extraction watermark. I do not assume zero is always an error because some feeds legitimately have no changes.

### 28. What would you improve before production?

**Model answer:** I would emit JSON logs to a centralized system, define stable metric names and units, propagate run IDs across services, add dashboards and actionable alerts, define SLOs, protect sensitive fields, set retention and access policies, and document runbooks. I would also test alert conditions and verify that metrics remain low-cardinality.

---

## Part 3 — Scenario questions with strong answers

### 29. The pipeline status is SUCCESS, but the target table is empty.

**Strong answer:** I treat this as a possible silent data failure. I inspect rows read, accepted, rejected and written, plus the source query window, API pagination, filters, target path and transaction result. I compare the run with historical counts and expected source activity. I do not close the incident until I explain the zero-row result and verify the target.

### 30. Rejection rate rises from 0.1% to 25%.

**Strong answer:** I alert based on the agreed quality threshold and investigate the affected run. I group rejects by reason, compare schema and source version, inspect sample records securely, and determine whether a producer change or transformation defect caused the increase. I quarantine invalid data and avoid publishing if the business quality gate is breached.

### 31. Logs contain a database password.

**Strong answer:** I treat it as a security incident. I restrict access to the affected logs, rotate the exposed credential, remove or redact the secret according to incident policy, inspect access history and fix the logging code. I prevent recurrence by using secret references and structured redaction. I do not simply delete the line and assume the credential is safe.

### 32. The dashboard shows 100% success, but users report stale data.

**Strong answer:** I check whether the success metric only measures job completion. I add or inspect freshness based on source event time and target publication time, check late-arriving data and source delivery delays, and verify the latest processed watermark. The pipeline may be technically successful while failing the business freshness objective.

### 33. Your metrics system becomes overloaded after adding run_id and customer_id labels.

**Strong answer:** I remove high-cardinality identifiers from metric labels. I keep bounded labels such as pipeline, stage and status, and put run_id or customer identifiers in searchable logs or traces with appropriate privacy controls. I review time-series volume and define metric naming and label standards.

### 34. A warning alert fires every five minutes for the same failed run.

**Strong answer:** I deduplicate or group alerts by pipeline and run ID, define a cooldown or repeat interval, and ensure the first alert contains actionable context and a runbook. I do not suppress genuinely new incidents. I measure alert noise and review thresholds with operators.

### 35. How would you set an SLO for a daily orders pipeline?

**Strong answer:** I first confirm the business deadline and acceptable failure rate. I define an SLI such as the percentage of scheduled runs that publish validated data before 7 AM. Then I set a target and measurement window, for example 99% over 30 days, and document exclusions and how failed or late runs count. I also monitor freshness and data-quality gates so a fast but incorrect run does not count as success.

---

## Part 4 — Practical revision checklist

- Explain logs, metrics and traces.
- Show how run_id connects stage events.
- Identify safe and unsafe log fields.
- Explain row counts and reconciliation.
- Calculate duration, throughput and freshness.
- Distinguish SLI, SLO and SLA.
- Explain alert fatigue and metric cardinality.
- Diagnose a success-with-zero-rows scenario.
- Walk through the JSON run summary and runbook.

## Part 5 — Self-assessment

| Skill | Score (0–2) |
|---|---:|
| Logs, metrics and traces | /2 |
| Run ID and structured logs | /2 |
| Security and redaction | /2 |
| Counts and reconciliation | /2 |
| Freshness and latency | /2 |
| SLI, SLO and SLA | /2 |
| Alerts and cardinality | /2 |
| Project incident diagnosis | /2 |

**13–16:** ready for junior-level discussion. **9–12:** repeat the observability workbook and scenario drills. **0–8:** revisit the class notes and build the run summary again.
