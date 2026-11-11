# Day 39: Pipeline Logging and Observability
**Phase:** 4 — Data Processing and Pipelines  
**Learning time:** 2–3 hours  
**Audience:** Complete beginners and freshers  
**Topic boundary:** These are class notes only. The project workbook and interview guide remain separate files.

> A pipeline can finish without an exception and still produce the wrong result. Logging and observability help a Data Engineer understand what happened, how much data moved, how long each stage took and whether the output is trustworthy. Today we learn to measure the pipeline, not merely print “success”.

## Learning outcomes

By the end of this lesson, you should be able to:
- explain logging, metrics, monitoring and observability;
- distinguish logs, metrics and traces;
- create and propagate a run ID;
- record stage start, completion and failure;
- understand structured logs;
- measure rows read, accepted, rejected and written;
- calculate duration and rejection rate;
- understand freshness, latency and throughput;
- distinguish SLA, SLO and alert threshold;
- design useful alerts without alert fatigue;
- understand metric cardinality;
- protect sensitive information in logs;
- build a basic run summary;
- investigate a failed or suspicious pipeline run.

## Today's flow

`Run starts -> Assign run ID -> Log stages -> Measure data -> Reconcile -> Evaluate thresholds -> Alert -> Investigate`

---

# 1. Why do Data Engineers need observability?

Imagine a daily pipeline normally processes 100,000 orders in five minutes.

Today it reports:

`Job completed successfully`

But the analytics dashboard contains zero orders.

The process may have exited normally while:
- the source query returned no rows;
- the date filter was wrong;
- the API returned only the first page;
- every record was rejected;
- the destination write was skipped;
- the output path was wrong.

A success message does not explain what happened.

Observability gives operators evidence about the internal state of a system through its outputs.

## 1.1 What questions should an operator answer?

For each run:
1. Which pipeline ran?
2. Which run instance is this?
3. When did it start?
4. When did it finish?
5. Which source was read?
6. How many records were read?
7. How many were accepted?
8. How many were rejected?
9. How many were written?
10. How long did each stage take?
11. Did retries occur?
12. Did reconciliation pass?
13. What failed?
14. Is the output fresh enough for the business?

If the system cannot answer these questions, troubleshooting becomes guesswork.

---

# 2. Understand logging

A log is a recorded event describing something that happened.

Examples:
- pipeline started;
- database connection opened;
- 5,000 rows fetched;
- 120 records rejected;
- output file published;
- checkpoint saved;
- API request timed out.

Logs help explain the sequence of events.

## 2.1 Print versus logging

Python's `print` displays a value to the current output stream.

Logging provides levels, timestamps, formatting and integration with log collection systems.

For a tiny learning example, print is fine. For a pipeline, structured logging is more useful.

## 2.2 Import logging

```python
import logging
```

The logging module is included with Python.

## 2.3 Configure a basic logger

```python
logging.basicConfig(
    level=logging.INFO,
    format="%(levelname)s %(message)s"
)
```

Explanation:
- `basicConfig` configures the default logging system.
- `level=logging.INFO` sets the minimum level displayed.
- `format` controls the text layout.
- `%(levelname)s` inserts the severity name.
- `%(message)s` inserts the log message.

## 2.4 Write an INFO log

```python
logging.info("Pipeline started")
```

Expected output resembles:

```text
INFO Pipeline started
```

The exact timestamp and formatting depend on configuration.

## 2.5 Why log events rather than random messages?

A useful log tells an operator:
- what happened;
- where it happened;
- which run it belongs to;
- whether it is expected or a problem.

Avoid vague logs such as:
- `here`;
- `done`;
- `something failed`.

Prefer:
- `source_extract_completed rows_read=5000`;
- `validation_completed accepted=4900 rejected=100`;
- `destination_write_failed batch_id=12 error_type=TimeoutError`.

---

# 3. Understand logging levels

| Level | Meaning | Example |
|---|---|---|
| DEBUG | Detailed diagnostic information | Current batch boundary |
| INFO | Normal progress | Extract stage completed |
| WARNING | Unexpected but recoverable condition | Rejection rate increased |
| ERROR | An operation failed | Database write failed |
| CRITICAL | Severe condition requiring urgent action | Pipeline cannot continue and data integrity is at risk |

## 3.1 DEBUG

Use DEBUG for details useful during diagnosis.

Do not enable highly verbose debug output permanently if it creates excessive volume or exposes sensitive values.

## 3.2 INFO

Use INFO for normal milestones:
- run started;
- source connected;
- page completed;
- output published.

## 3.3 WARNING

Use WARNING when the pipeline can continue but an operator should notice:
- unusually high rejection rate;
- missing optional feed;
- retry occurred;
- processing slower than normal.

## 3.4 ERROR

Use ERROR when a stage or operation fails.

Include safe context such as run ID, stage and error type.

## 3.5 CRITICAL

Use CRITICAL for severe failures that prevent safe continuation or threaten data integrity.

Do not mark every ordinary validation failure as critical.

---

# 4. Understand a run ID

A run ID uniquely identifies one execution of a pipeline.

Example:
- pipeline: daily_orders;
- run_id: run-20261111-001;
- start time: 02:00;
- status: SUCCESS.

If the pipeline runs twice, each execution should have a different run ID.

## 4.1 Why is run ID important?

Without a run ID, logs from two overlapping executions may be mixed together.

A run ID allows the operator to:
- filter logs;
- connect stage metrics;
- find the correct output;
- inspect retries;
- investigate one incident.

## 4.2 Create a UUID run ID

```python
import uuid

run_id = str(uuid.uuid4())

print(run_id)
```

The output is a unique-looking identifier. Its exact value changes on each execution.

## 4.3 Reuse the same run ID across stages

The run ID should be created once at the start of the run and passed to every stage.

Do not create a new run ID for each log line. That would make one run appear to be many unrelated runs.

## 4.4 Run ID and batch ID

A run can contain multiple batches.

Example:
- run_id identifies the whole daily execution;
- batch_id identifies one unit within that run.

Together they allow an operator to find one failed batch within a larger run.

---

# 5. Log the pipeline lifecycle

A useful pipeline records key transitions.

Example lifecycle:

`STARTED -> EXTRACTING -> VALIDATING -> WRITING -> RECONCILING -> SUCCEEDED`

A failure may move the run to FAILED or PARTIAL according to the pipeline contract.

## 5.1 Log run start

```python
logging.info("run_id=%s event=run_started", run_id)
```

The format placeholders receive the run ID.

## 5.2 Log stage completion

```python
logging.info(
    "run_id=%s stage=extract status=completed rows_read=%d",
    run_id,
    5000
)
```

This records the stage, status and number of rows read.

## 5.3 Log stage failure

```python
logging.error(
    "run_id=%s stage=load status=failed error_type=%s",
    run_id,
    "TimeoutError"
)
```

This captures safe diagnostic context. In a real exception handler, use the actual exception type and a redacted message.

## 5.4 Avoid logging the entire data record

A record may contain:
- customer email;
- address;
- phone number;
- payment details;
- access token.

Log record identifiers only when policy allows, and prefer aggregate counts or masked references.

---

# 6. Understand structured logging

A plain text log is readable but can be difficult for machines to filter reliably.

Structured logging stores named fields.

Example JSON log:

```json
{
  "timestamp": "2026-11-11T02:00:00Z",
  "level": "INFO",
  "run_id": "run-001",
  "pipeline": "daily_orders",
  "stage": "extract",
  "rows_read": 5000,
  "status": "SUCCESS"
}
```

Each value has a field name.

A log platform can filter:
- all logs for run-001;
- all ERROR events;
- all events from daily_orders;
- all extraction stages with rows_read equal to zero.

## 6.1 Why JSON logs help

They support:
- filtering;
- aggregation;
- dashboards;
- alert rules;
- correlation across services.

## 6.2 Structured log concept in Python

```python
import json

log_record = {
    "run_id": run_id,
    "pipeline": "daily_orders",
    "stage": "extract",
    "rows_read": 5000,
    "status": "SUCCESS"
}

print(json.dumps(log_record))
```

Expected output is one JSON-formatted line containing the supplied fields.

This demonstrates serialization. Production systems should use an approved structured logging handler rather than relying only on print.

---

# 7. Understand metrics

A metric is a numeric measurement collected over time.

Examples:
- rows read;
- rows written;
- rejected records;
- request latency;
- job duration;
- retry count;
- bytes processed;
- output freshness.

Metrics help detect changes and trends.

## 7.1 Counter

A counter measures a value that increases over time, such as:
- records processed;
- errors observed;
- API requests sent.

## 7.2 Gauge

A gauge represents a value that can go up or down, such as:
- current queue depth;
- active worker count;
- current memory usage.

## 7.3 Histogram

A histogram records distributions, such as:
- request duration;
- batch processing time;
- record size.

It helps answer whether a few operations are very slow.

## 7.4 Metric versus log

A metric says:

`rows_rejected = 100`

A log may explain:

`run_id=run-001 stage=validation rejected=100 reason=invalid_amount`

Metrics show trends. Logs provide event details.

---

# 8. Measure record counts

A pipeline should measure records at important boundaries.

Common counters:
- input_rows;
- rows_read;
- rows_validated;
- rows_accepted;
- rows_rejected;
- rows_written;
- rows_updated;
- rows_inserted.

Do not use one generic `count` for every stage. Name the measurement clearly.

## 8.1 Example counts

```python
rows_read = 1000
rows_accepted = 970
rows_rejected = 30
rows_written = 970

print(rows_read)
print(rows_accepted)
print(rows_rejected)
print(rows_written)
```

The values are a teaching fixture.

## 8.2 Reconciliation metric

```python
accounted_rows = rows_accepted + rows_rejected

if rows_read == accounted_rows:
    reconciliation_status = "PASSED"
else:
    reconciliation_status = "FAILED"

print(reconciliation_status)
```

Expected output:

```text
PASSED
```

The check proves the read rows were classified. It does not prove the output file or destination contains the correct values.

## 8.3 Output reconciliation

```python
if rows_written == rows_accepted:
    print("Write count matches accepted count")
else:
    print("Write count mismatch")
```

This checks the counts. Also validate keys and important values.

---

# 9. Measure rejection rate

Rejection rate helps identify data-quality problems.

For the teaching example:
- rows read = 1,000;
- rows rejected = 30.

The rejection rate is 3 percent.

## 9.1 Calculate safely

```python
if rows_read > 0:
    rejection_rate = (rows_rejected / rows_read) * 100
else:
    rejection_rate = 0

print(rejection_rate)
```

Expected output:

```text
3.0
```

If no rows were read, the calculation must avoid division by zero. Whether a zero-row run is acceptable is a separate business rule.

## 9.2 Why monitor rejection rate?

A sudden increase may indicate:
- source schema changed;
- a new status value appeared;
- a producer sent malformed data;
- transformation logic is wrong;
- source quality deteriorated.

The metric tells us something changed. Logs and rejected records help explain why.

---

# 10. Measure duration and throughput

## 10.1 What is duration?

Duration is the time taken by a stage or full run.

A pipeline may be successful but too slow to meet the business deadline.

## 10.2 Measure duration with time.perf_counter

```python
import time

start_time = time.perf_counter()

total = 0
for number in range(100000):
    total = total + number

end_time = time.perf_counter()

duration_seconds = end_time - start_time

print(duration_seconds)
```

The exact duration depends on the computer and current load.

`perf_counter` is useful for measuring elapsed time within one process.

## 10.3 Throughput

Throughput measures how much work is processed per unit of time.

Example:
- 10,000 rows;
- 20 seconds;
- 500 rows per second.

## 10.4 Calculate throughput

```python
rows_processed = 10000
duration_seconds = 20

if duration_seconds > 0:
    rows_per_second = rows_processed / duration_seconds
else:
    rows_per_second = 0

print(rows_per_second)
```

Expected output:

```text
500.0
```

Throughput alone does not prove quality. A pipeline can process incorrect data very quickly.

---

# 11. Understand freshness and latency

## 11.1 Freshness

Freshness describes how recently the available data reflects the source.

Example:
- latest source event: 02:10;
- latest warehouse event: 02:15;
- data freshness lag: 5 minutes.

## 11.2 Latency

Latency is the time between an event or request and its result.

For a data pipeline, define which points are being compared:
- source event time to destination availability;
- job start to job finish;
- API request to response.

## 11.3 Why freshness matters

A daily dashboard may be acceptable with data updated by 07:00.

A fraud detection system may need data within seconds.

The business use case defines the required freshness.

## 11.4 Do not confuse job completion with freshness

A job can finish successfully while processing an old source partition.

Measure the latest business timestamp or source watermark, not only job end time.

---

# 12. Understand traces and correlation

A trace follows one operation across multiple components.

Example:
`Orchestrator -> Python extractor -> API provider -> Database loader`

A trace ID can connect events from different services.

## 12.1 Run ID versus trace ID

A run ID identifies a pipeline execution.

A trace ID identifies a connected request or distributed operation. One pipeline run may contain multiple traces.

A span ID can identify one operation within a trace.

For beginner batch pipelines, a run ID is a useful first correlation field. Distributed systems may need trace IDs and spans.

## 12.2 Why correlation matters

If the API extractor and database loader use unrelated identifiers, an operator may struggle to connect the API response to the destination write.

Propagate correlation identifiers across stages.

---

# 13. Understand SLA, SLO and alert thresholds

## 13.1 SLA

A Service Level Agreement is an agreement with a customer or business stakeholder about a service level.

Example:
- daily report must be available by 07:00.

## 13.2 SLO

A Service Level Objective is an internal target for service performance.

Example:
- 99 percent of daily pipeline runs complete before 06:30 over a defined period.

## 13.3 Alert threshold

An alert threshold is a condition that triggers a notification.

Example:
- job is still running at 06:15;
- rejection rate exceeds 5 percent;
- source row count is zero when data is expected;
- freshness lag exceeds 30 minutes.

These concepts are related but not interchangeable.

## 13.4 Define thresholds from business needs

Do not invent an alert threshold because it looks round.

Use:
- historical baseline;
- business deadline;
- acceptable quality level;
- operational response time;
- cost of false alerts.

---

# 14. Design useful alerts

An alert should tell an operator:
- what happened;
- how serious it is;
- which pipeline/run is affected;
- what business data may be affected;
- where to investigate;
- what action is expected.

## 14.1 High rejection rate

Condition:
- rejection rate exceeds approved threshold.

Possible cause:
- schema drift;
- source data quality issue;
- transformation defect.

## 14.2 Zero rows

Condition:
- rows_read = 0 when a delivery is expected.

Possible cause:
- missing source file;
- incorrect filter;
- API pagination failure;
- source delay.

Zero rows may be valid for some feeds. The expected-volume contract matters.

## 14.3 Long duration

Condition:
- run duration exceeds its historical or agreed threshold.

Possible cause:
- source slowdown;
- increased data volume;
- query plan change;
- destination bottleneck.

## 14.4 Alert fatigue

If every harmless warning sends an urgent alert, operators may stop responding.

Use:
- severity levels;
- deduplication;
- thresholds;
- grouping;
- maintenance windows;
- clear ownership.

---

# 15. Understand metric cardinality

Cardinality means the number of distinct values a metric label can take.

Low-cardinality labels:
- pipeline name;
- stage;
- environment;
- status.

High-cardinality labels:
- customer_id;
- order_id;
- email address;
- unique request ID.

Putting every order_id into metric labels can create millions of time series.

Use metrics for bounded dimensions. Put detailed identifiers in appropriately protected logs or traces.

---

# 16. Protect sensitive data in logs

Logs often have broad access and long retention.

Never log:
- passwords;
- API tokens;
- database connection strings;
- full payment card details;
- unnecessary personal data;
- private keys.

Use:
- redaction;
- masking;
- field allowlists;
- access controls;
- retention limits.

A log line should contain enough information to diagnose the issue without exposing the record's sensitive content.

---

# 17. Build a simple run summary

## 17.1 Define run measurements

```python
run_summary = {
    "run_id": run_id,
    "pipeline": "daily_orders",
    "rows_read": 1000,
    "rows_accepted": 970,
    "rows_rejected": 30,
    "rows_written": 970,
    "status": "SUCCESS"
}

print(run_summary)
```

This dictionary represents one run summary.

## 17.2 Add rejection rate

```python
run_summary["rejection_rate"] = (
    run_summary["rows_rejected"] /
    run_summary["rows_read"]
) * 100

print(run_summary["rejection_rate"])
```

The calculation assumes rows_read is greater than zero. A production implementation must handle zero rows.

## 17.3 Add a duration field

```python
run_summary["duration_seconds"] = 42.5

print(run_summary)
```

A run summary should be stored durably, not only printed and lost when the process ends.

## 17.4 Save a JSON summary

```python
import json

with open("run_summary.json", "w", encoding="utf-8") as file:
    json.dump(run_summary, file, indent=2)

print("Run summary saved")
```

The summary can later be loaded by a dashboard or monitoring process.

---

# 18. Investigate an unhealthy run

Suppose a pipeline usually reads 100,000 rows. Today's run reads 2,000 and completes in 20 seconds.

The run status is SUCCESS.

Do not accept success without investigation.

Check:
1. source delivery count;
2. query date range;
3. API page count;
4. source maximum timestamp;
5. accepted and rejected counts;
6. output row count;
7. previous run trend;
8. source schema changes;
9. run ID and stage logs.

A sudden count drop may be a business event or a pipeline defect. Use evidence to distinguish them.

---

# 19. Common observability mistakes

## Mistake 1 — Log only “success”

**Risk:** Operators cannot explain zero rows or incomplete output.

**Correction:** Log stage, counts, duration and reconciliation status.

## Mistake 2 — No run ID

**Risk:** Concurrent runs cannot be separated.

**Correction:** Create one run ID and propagate it.

## Mistake 3 — Log every record

**Risk:** Huge log volume and possible sensitive data exposure.

**Correction:** Log aggregate metrics and selected safe identifiers.

## Mistake 4 — Use customer ID as a metric label

**Risk:** Very high cardinality.

**Correction:** Keep metric labels bounded; use protected logs for detail.

## Mistake 5 — Alert on every warning

**Risk:** Alert fatigue.

**Correction:** Set severity, thresholds and ownership.

## Mistake 6 — Measure duration only

**Risk:** A fast job may have processed zero rows.

**Correction:** Measure counts, freshness and quality as well.

## Mistake 7 — Trust process exit code alone

**Risk:** The program may exit successfully with incorrect data.

**Correction:** Include reconciliation and business checks.

## Mistake 8 — Log secrets

**Risk:** Credential exposure.

**Correction:** Redact and minimize sensitive fields.

---

# 20. Practical exercises

## Exercise 1 — Choose the right log level

Pipeline started normally.

Expected level: INFO.

A retry occurred but the pipeline recovered.

Expected level: WARNING or an equivalent retry event, depending on the logging policy.

A destination write failed and the run stopped.

Expected level: ERROR.

## Exercise 2 — Calculate rejection rate

Rows read = 2,000. Rows rejected = 40.

Expected rate: 2 percent.

## Exercise 3 — Calculate throughput

Rows processed = 12,000. Duration = 60 seconds.

Expected throughput: 200 rows per second.

## Exercise 4 — Identify missing observability

A job reports SUCCESS but does not report rows_read.

Expected answer: the operator cannot determine whether the job processed the expected amount of data.

## Exercise 5 — Identify cardinality risk

Should every unique order_id be a metric label?

Expected answer: no. It creates very high cardinality.

## Exercise 6 — Explain run ID

Two executions overlap. Why is a run ID necessary?

Expected answer: it allows logs, metrics and outputs to be associated with the correct execution.

## Exercise 7 — Explain freshness

A job finishes at 07:00 but the newest source event is from yesterday.

Expected answer: job completion does not prove data freshness. Inspect the latest business timestamp and source boundary.

## Exercise 8 — Design an alert

A daily feed normally contains at least 10,000 records. Today's count is zero.

Expected reasoning: alert if the feed contract says a delivery is expected. Confirm source arrival, filters and pagination.

---

# 21. Debugging checklist

When an operator reports a pipeline issue:

1. Find the pipeline name.
2. Find the run ID.
3. Check run start and end times.
4. Check stage transitions.
5. Inspect source and destination names.
6. Compare rows read, accepted, rejected and written.
7. Check reconciliation status.
8. Inspect duration and throughput.
9. Check latest source and destination timestamps.
10. Inspect retry count and failure type.
11. Review safe error details.
12. Check whether alerts fired.
13. Compare with historical runs.
14. Confirm no secrets or sensitive values were exposed.
15. Record root cause and corrective action.

---

# 22. Day 39 revision checklist

You should be able to explain:
- logging;
- log levels;
- metrics;
- monitoring;
- observability;
- run ID;
- batch ID;
- structured logs;
- counters, gauges and histograms;
- rows read, accepted, rejected and written;
- rejection rate;
- duration and throughput;
- freshness and latency;
- trace ID and span;
- SLA and SLO;
- alert threshold;
- alert fatigue;
- metric cardinality;
- sensitive data redaction;
- run summary;
- reconciliation-based health checks.

---

# 23. Key takeaways

1. A successful process exit does not prove correct data.
2. Logs explain events; metrics measure behavior; traces connect operations.
3. Create one run ID and propagate it.
4. Record stage transitions and failure context.
5. Measure rows read, accepted, rejected and written.
6. Measure duration, throughput and freshness.
7. Reconcile data before marking a run successful.
8. Define SLA, SLO and alert thresholds clearly.
9. Avoid high-cardinality metric labels.
10. Never log secrets or unnecessary personal data.
11. Use alerts that lead to a clear operational action.
12. Compare current runs with historical behavior.

## Phase 4 class-notes completion checkpoint

The Phase 4 learning sequence now covers:
- Day 30: ETL and Data Extraction;
- Day 31: File-Based Data Ingestion;
- Day 32: API-Based Data Ingestion;
- Day 33: Database Ingestion;
- Day 34: Data Transformation and Business Rules;
- Day 35: Data Cleaning and Schema Evolution;
- Day 36: Batch Processing and Partitioning;
- Day 37: Incremental Loading and Upserts;
- Day 38: Pipeline Reliability and Failure Handling;
- Day 39: Pipeline Logging and Observability.

This list describes the course sequence, not a claim that every file has been runtime-tested.

**End of Day 39 class notes.**