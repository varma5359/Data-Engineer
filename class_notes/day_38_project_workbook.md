# Day 38 Project Workbook — Reliable Pipeline Simulator

## Goal
Demonstrate safe replay, idempotent destination writes, bounded retry and checkpoint discipline.

## 1. Define source and target
```python
batch = [
 {"event_id":"E1","order_id":501,"amount":100},
 {"event_id":"E2","order_id":502,"amount":200},
 {"event_id":"E3","order_id":503,"amount":300}
]
target = {}
checkpoint = 0
```

## 2. Idempotent write by stable event key
```python
for event in batch:
    target[event["event_id"]] = event
print(len(target))
```
Expected 3. Re-run the same cell; expected remains 3, not 6.

## 3. Simulate crash after write, before checkpoint
```python
checkpoint = 0
print("target rows", len(target), "checkpoint", checkpoint)
```
On restart, replay the batch. Stable event IDs prevent duplicate keys in this demonstration.

## 4. Replay and verify
```python
for event in batch:
    target[event["event_id"]] = event
expected = {x["event_id"] for x in batch}
assert expected.issubset(set(target))
checkpoint = 3
print("rows", len(target), "checkpoint", checkpoint)
```
Expected rows 3, checkpoint 3. Production systems should commit target and checkpoint atomically where possible, or use a durable manifest/recovery protocol.

## 5. Retry decision table

| Failure | Retry? | Action |
|---|---|---|
| Temporary timeout | Usually | bounded retry with backoff |
| HTTP 429 | Yes, respect Retry-After | delay and retry |
| Invalid schema | No blind retry | quarantine and alert owner |
| Permission denied | No | fix credentials/permissions securely |
| Database deadlock | Often | retry transaction with limit |
| Disk full | Not until repaired | stop, alert, recover capacity |

## 6. Recovery runbook
Identify run/batch; inspect committed target state; locate last safe checkpoint; correct transient/permanent cause; replay bounded unit; reconcile unique keys and counts; mark recovered only after verification.

## Acceptance
Replay does not duplicate; checkpoint never moves ahead of durable output; transient/permanent failures are distinguished; retry count is bounded; recovery evidence is recorded.
