
# Day 38 Project Workbook — TechStore Reliable Pipeline Simulator

**Goal:** Simulate transient failures, bounded retries, idempotent writes, partial failure, checkpoint discipline and quarantine.  
**Prerequisites:** Python loops, dictionaries, exceptions and Day 37. No external package required.

## 1. Why build a failure simulator?

A pipeline must be safe when a source is unavailable, a job crashes halfway or an operator reruns it. A transient failure may recover after a retry. A permanent data error will not be fixed by repeating the same request.

## 2. Create a batch and target

~~~python
batch = [
    {"event_id": "E1", "order_id": 501, "amount": 100},
    {"event_id": "E2", "order_id": 502, "amount": 200},
    {"event_id": "E3", "order_id": 503, "amount": 300}
]
target = {}
checkpoint = 0
print("Incoming events:", len(batch))
print("Initial target:", len(target))
print("Initial checkpoint:", checkpoint)
~~~

Expected: 3 events, 0 target rows, checkpoint 0.

## 3. Use a stable key for idempotent writes

~~~python
for event in batch:
    target[event["event_id"]] = event.copy()
print("Rows after first write:", len(target))
~~~

Expected: 3. Run the same cell again:

~~~python
for event in batch:
    target[event["event_id"]] = event.copy()
print("Rows after replay:", len(target))
~~~

Expected: still 3. A production database must enforce event_id with a primary key or unique constraint. A Python dictionary alone is only a teaching simulation.

## 4. Simulate a transient read failure

~~~python
attempts = 0
max_attempts = 3
source_available_after = 2
read_succeeded = False

while attempts < max_attempts and not read_succeeded:
    attempts += 1
    print("Attempt", attempts)
    if attempts < source_available_after:
        print("Temporary source error")
    else:
        read_succeeded = True
        print("Source read succeeded")
print("Attempts used:", attempts)
~~~

Expected: attempt 1 temporary error; attempt 2 succeeds; attempts used 2. The loop is bounded and cannot retry forever.

## 5. Calculate exponential backoff

~~~python
base_delay = 2
maximum_delay = 20
for retry_number in range(1, 5):
    delay = min(base_delay * (2 ** (retry_number - 1)), maximum_delay)
    print("Retry", retry_number, "delay seconds:", delay)
~~~

Expected delays: 2, 4, 8, 16 seconds. Production systems should add jitter and a total time budget. This cell calculates delays; it does not sleep.

## 6. Do not retry permanent data errors

~~~python
event = {"event_id": "E4", "amount": "not-a-number"}
if not isinstance(event["amount"], (int, float)):
    print("Permanent data error: amount must be numeric")
else:
    print("Record is valid")
~~~

Expected: permanent data error. Quarantine unchanged invalid input; do not retry it endlessly.

## 7. Advance checkpoint only after verification

~~~python
target = {}
checkpoint = 0
batch_end = 3

for event in batch:
    target[event["event_id"]] = event.copy()

target_verified = len(target) == len(batch)
if target_verified:
    checkpoint = batch_end

print("Target verified:", target_verified)
print("Checkpoint:", checkpoint)
~~~

Expected: True and checkpoint 3. If a write fails halfway, checkpoint must remain 0.

## 8. Simulate crash and recovery

~~~python
target = {}
checkpoint = 0
for event in batch[:2]:
    target[event["event_id"]] = event.copy()
print("Rows after simulated crash:", len(target))
print("Checkpoint after crash:", checkpoint)

for event in batch:
    target[event["event_id"]] = event.copy()

target_verified = len(target) == 3
if target_verified:
    checkpoint = 3
print("Rows after recovery:", len(target))
print("Checkpoint after recovery:", checkpoint)
~~~

Expected: after crash 2 rows/checkpoint 0; after recovery 3 rows/checkpoint 3. Stable keys make replay safe.

## 9. Quarantine a bad record

~~~python
incoming = [
    {"event_id": "E10", "amount": 50},
    {"event_id": "E11", "amount": -5},
    {"event_id": "E12", "amount": 80}
]
accepted = []
rejected = []
for row in incoming:
    if row["amount"] < 0:
        rejected.append({"record": row, "reason": "negative_amount"})
    else:
        accepted.append(row)
print("Accepted:", len(accepted))
print("Rejected:", len(rejected))
print("Reason:", rejected[0]["reason"])
~~~

Expected: 2 accepted, 1 rejected, reason negative_amount. Whether one bad record fails the entire batch depends on the data contract.

## 10. Failure decision table

| Failure | Retry? | Recovery |
|---|---|---|
| Temporary connection reset | Usually, bounded | Backoff and retry |
| HTTP 429 | Respect provider delay | Retry after Retry-After |
| Invalid amount | No unchanged retry | Quarantine |
| Missing required column | No | Fail batch and contact source owner |
| Destination unavailable | Bounded if operation safe | Retry or replay idempotently |
| Checkpoint failed after target commit | Retry checkpoint/replay | Keep old checkpoint until persisted |
| Duplicate event | Not a new insert | Unique key/upsert and investigate |

## 11. Failure drills

1. Set max_attempts to 1; show the job stops after one failure.
2. Set source_available_after to 5 and max_attempts to 3; show bounded failure.
3. Replay the batch; target remains three unique event IDs.
4. Crash after two writes; checkpoint remains unchanged.
5. Change E2 amount to a negative value; quarantine it.
6. Explain why retrying an external payment POST without an idempotency key may create duplicate side effects.

## 12. Production design rules

- Configure request and connection timeouts.
- Retry only classified transient errors.
- Use bounded attempts, exponential backoff, jitter and retry budgets.
- Use stable event keys and database uniqueness constraints.
- Commit destination before advancing checkpoint.
- Use transactions for database writes.
- Write files to temporary staging, validate and then publish.
- Prevent concurrent workers from advancing the same checkpoint.
- Keep rejected records separate and avoid logging secrets.

## 13. Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Endless retries | No attempt/time limit | Add bounds |
| Duplicate records | No idempotency key | Stable unique key and upsert |
| Lost records after crash | Checkpoint advanced too early | Advance only after durable output |
| One bad row breaks all work | Record/batch errors not distinguished | Quarantine if contract allows |
| Service overload on recovery | Synchronized immediate retries | Backoff with jitter |

## 14. Acceptance checklist

- [ ] Transient and permanent errors distinguished.
- [ ] Retry bounded.
- [ ] Backoff calculated.
- [ ] Stable idempotency key used.
- [ ] Replay creates no duplicates.
- [ ] Partial failure leaves checkpoint unchanged.
- [ ] Recovery reprocesses incomplete range.
- [ ] Bad records quarantined with reasons.
- [ ] Can explain at-least-once and why exactly-once effects require end-to-end guarantees.

**Project explanation:** "I simulated bounded retries, backoff, idempotent writes, partial failure and recovery. The checkpoint stays behind durable output, incomplete work can be replayed and invalid records are quarantined."
