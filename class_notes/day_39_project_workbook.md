
# Day 39 Project Workbook — TechStore Pipeline Observability Tool

**Goal:** Add a run ID, structured stage events, processing metrics, reconciliation, duration/freshness checks and alert decisions.  
**Prerequisites:** Python dictionaries, JSON and basic file handling. Standard library only.

## 1. Business problem

A pipeline prints SUCCESS, but the dashboard is empty. A success message does not tell us whether the source returned zero rows, every record was rejected, an API page was missed, output is incomplete or data is stale.

Observability should answer: what happened, how much data moved, how long it took and whether the output is trustworthy.

## 2. Create the workspace

~~~bat
mkdir day39_observability_lab
cd day39_observability_lab
~~~

Create day39_observability.ipynb.

## 3. Create a run ID and context

~~~python
from datetime import datetime, timezone
from uuid import uuid4

run_id = str(uuid4())
started_at = datetime.now(timezone.utc)
run = {
    "run_id": run_id,
    "pipeline_name": "techstore_orders",
    "status": "RUNNING",
    "started_at": started_at.isoformat()
}
print("Pipeline:", run["pipeline_name"])
print("Status:", run["status"])
print("Run ID created:", bool(run_id))
~~~

Expected: pipeline techstore_orders, status RUNNING, Run ID created True. UUID and timestamps vary. Propagate the same run ID to every stage so logs can be correlated.

## 4. Configure logging

~~~python
import logging
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(message)s"
)
logger = logging.getLogger("techstore_pipeline")
logger.info("Pipeline started run_id=%s", run_id)
logger.warning("One source record requires review run_id=%s", run_id)
~~~

INFO and WARNING messages appear. DEBUG is hidden at INFO level. ERROR indicates an operation failed. CRITICAL indicates a severe condition.

## 5. Record stage events

~~~python
stage_events = []
stage_events.append({
    "run_id": run_id, "stage": "extract",
    "status": "SUCCESS", "rows_read": 1000
})
stage_events.append({
    "run_id": run_id, "stage": "validate",
    "status": "SUCCESS", "accepted_rows": 970,
    "rejected_rows": 30
})
stage_events.append({
    "run_id": run_id, "stage": "write",
    "status": "SUCCESS", "rows_written": 970
})
for event in stage_events:
    print(event)
~~~

The same run ID connects the stages. A production pipeline emits these events as stages start, complete or fail.

## 6. Calculate metrics

~~~python
input_rows = 1000
accepted_rows = 970
rejected_rows = 30
written_rows = 970
retry_count = 1
duration_seconds = 120

rejection_rate = rejected_rows / input_rows * 100 if input_rows else 0
throughput = input_rows / duration_seconds if duration_seconds else 0
reconciliation_difference = input_rows - accepted_rows - rejected_rows

metrics = {
    "input_rows": input_rows,
    "accepted_rows": accepted_rows,
    "rejected_rows": rejected_rows,
    "written_rows": written_rows,
    "retry_count": retry_count,
    "rejection_rate_percent": round(rejection_rate, 2),
    "throughput_rows_per_second": round(throughput, 2),
    "reconciliation_difference": reconciliation_difference
}
print(metrics)
~~~

Expected: rejection rate 3.0%, throughput 8.33 rows/second, reconciliation difference 0. Thirty of 1,000 rows were rejected; 970 accepted rows were written. Also verify key uniqueness and business totals: a zero count difference alone is not sufficient.

## 7. Evaluate health rules

~~~python
alerts = []
if input_rows == 0:
    alerts.append("ZERO_SOURCE_ROWS")
if reconciliation_difference != 0:
    alerts.append("RECONCILIATION_FAILED")
if rejection_rate > 5:
    alerts.append("HIGH_REJECTION_RATE")
if duration_seconds > 300:
    alerts.append("PIPELINE_TOO_SLOW")

status = "ATTENTION_REQUIRED" if alerts else "HEALTHY"
print("Status:", status)
print("Alerts:", alerts)
~~~

Expected for this fixture: HEALTHY and an empty alerts list. The 5% and 300-second limits are teaching examples, not universal production thresholds. Business owners define acceptable limits.

## 8. Save a JSON run summary

~~~python
import json
finished_at = datetime.now(timezone.utc)
summary = {
    "run_id": run_id,
    "pipeline_name": "techstore_orders",
    "status": status,
    "started_at": started_at.isoformat(),
    "finished_at": finished_at.isoformat(),
    "metrics": metrics,
    "alerts": alerts
}
with open("run_summary.json", "w", encoding="utf-8") as file:
    json.dump(summary, file, indent=2)
print("Saved: run_summary.json")
print(json.dumps(summary, indent=2))
~~~

UUID and timestamps vary; metric values remain stable for this fixture. Protect run artifacts according to data retention policy.

## 9. Investigate an unhealthy run

~~~python
written_rows = 900
if written_rows != accepted_rows:
    print("ALERT: written row count does not match accepted row count")
else:
    print("Write count matches accepted count")
~~~

Expected: alert appears. A process can exit successfully but still write incomplete output. Reconciliation belongs in pipeline health checks.

Now set input_rows to zero. Is that expected on a holiday, or does it indicate a broken source filter? Monitoring thresholds need business context.

## 10. Logs, metrics, traces and alerts

| Signal | Question | Example |
|---|---|---|
| Logs | What happened? | Extraction started; write failed |
| Metrics | How much/how long? | Rows, duration, retries |
| Traces | Which stages/services did one run traverse? | API -> transform -> database |
| Alerts | Which condition needs action? | Freshness breach or reconciliation failure |

Freshness measures how old the newest published data is. Latency measures elapsed time between event/source availability and downstream availability. A fast job can still load stale data.

## 11. Security and alert quality

Never log passwords, API tokens, connection strings, payment details or unnecessary personal data. Use safe identifiers and error categories. Avoid alerting on every harmless warning; alerts need an owner and an action.

Do not use customer IDs or order IDs as metric labels. High-cardinality labels create too many time series and may expose sensitive information.

## 12. Failure drills

1. Set input_rows to zero and inspect the alert.
2. Set rejected_rows to 100; confirm 10% rejection triggers the alert.
3. Set written_rows below accepted_rows; confirm reconciliation alert.
4. Set duration_seconds to 600; confirm slow-pipeline alert.
5. Remove run_id from one stage; explain why correlation is harder.
6. Add customer_id as a metric label; explain cardinality and privacy risk.

## 13. Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| No INFO logs | Logger level/handler | Check active logging configuration |
| Events cannot be tied together | Run ID not propagated | Pass one run ID to every stage |
| Success but empty output | Only exit status checked | Add counts, freshness and reconciliation |
| Too many metric series | High-cardinality labels | Use bounded labels such as pipeline, stage and status |
| Secret in logs | Raw request/exception content | Redact and log safe error categories |

## 14. Acceptance checklist

- [ ] Each run has a unique run ID.
- [ ] Every stage records status.
- [ ] Appropriate log levels are used.
- [ ] Input, accepted, rejected and written counts are measured.
- [ ] Rejection rate and throughput are correct.
- [ ] Reconciliation is checked.
- [ ] Zero-row, slow-run and high-rejection alerts considered.
- [ ] JSON summary saved.
- [ ] Sensitive values are not logged.
- [ ] Can explain logs, metrics, traces, monitoring and observability.

**Project explanation:** "I instrumented an order pipeline with a run ID and stage events, measured row counts and throughput, checked reconciliation, evaluated alert rules and saved a JSON run summary."
