# Day 39 Project Workbook — Pipeline Observability Tool

## Goal
Create structured run events and operational metrics, validate counts, calculate rejection rate and design useful alerts.

## 1. Create a run
```python
from datetime import datetime, timezone
import json
run_id = "run-20260110-001"
started = datetime.now(timezone.utc)
metrics = {"input_rows":1000,"accepted_rows":970,"rejected_rows":30,
           "written_rows":970,"retry_count":1,"status":"SUCCESS"}
```

## 2. Validate counts
```python
assert metrics["input_rows"] == metrics["accepted_rows"] + metrics["rejected_rows"]
assert metrics["accepted_rows"] == metrics["written_rows"]
print("Reconciliation passed")
```
Expected: Reconciliation passed.

## 3. Create a structured log
```python
event = {
 "timestamp": started.isoformat(), "level":"INFO", "run_id":run_id,
 "stage":"write_curated", **metrics
}
print(json.dumps(event, indent=2))
```
Timestamp varies. Structured fields allow searching/grouping by run_id, stage and status. Never log credentials, tokens or sensitive personal information.

## 4. Calculate rejection rate
```python
rate = metrics["rejected_rows"] / metrics["input_rows"] * 100
print(round(rate, 2))
```
Expected: 3.0. Compare with baseline and approved threshold; 3% is not inherently good or bad without context.

## 5. Record a failure
```python
failure = {"run_id":"run-20260110-002","stage":"read_source",
           "error_type":"TimeoutError","retryable":True,"attempt":1}
print(failure)
```
A transient timeout may be retried. A schema violation normally needs source/code correction, not infinite retries.

## 6. Design alert rules
For each alert state condition, severity, owner, run link, impact and action. Examples: no successful daily run by 07:00; freshness lag exceeds agreed limit; rejection rate exceeds baseline threshold; repeated retries; reconciliation mismatch.

## Acceptance
Provide one success event, one failure event, row-count metrics, rejection rate, alert rules, and a short incident explanation. Show how run_id connects logs and metrics.
