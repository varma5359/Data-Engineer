# Day 62 Interview Guide — Airflow Scheduling and Reliability

## 1. What is a DAG run?

A DAG run is one execution of a DAG for a particular scheduled data interval or manual trigger. It contains task instances for the tasks in that workflow.

## 2. What is a task instance?

A task instance is the execution of one task within a particular DAG run. The same task can have separate instances across multiple runs.

## 3. What is a logical date?

A logical date identifies the scheduling period associated with a DAG run. It is not necessarily the actual wall-clock time when the task begins executing.

## 4. What is a data interval?

A data interval is the period of data associated with a scheduled DAG run. For a daily workflow, a run commonly processes the previous completed day. The task should use the run's interval rather than assuming the current system date is the business date.

## 5. Why does a daily DAG usually run after its interval ends?

A daily report often needs a complete day's data. Airflow generally schedules the run after the relevant interval has closed, although actual execution depends on scheduler and executor capacity.

## 6. What is catchup?

Catchup controls whether Airflow creates runs for missed historical scheduled intervals from the DAG's start boundary. It must be configured carefully because a long pause can create many runs.

## 7. What is a backfill?

A backfill is a deliberate reprocessing of historical intervals. It should have a defined date range, available source data, idempotent output, capacity planning and reconciliation.

## 8. What is the difference between catchup and backfill?

Catchup is automatic scheduling behavior for missed intervals. Backfill is an intentional operational action to run historical periods, typically for recovery or corrected logic.

## 9. What is a retry?

A retry is another task attempt after a failure. It is useful for temporary issues such as a short network timeout. It does not fix permanent problems such as invalid schema or incorrect permissions.

## 10. What is retry_delay?

retry_delay defines how long Airflow waits before another attempt. It should be selected based on the failure type and source-system recovery behavior.

## 11. What is exponential backoff?

Exponential backoff increases the delay between successive retry attempts. It can reduce repeated pressure on an unavailable service. The exact delay and caps depend on Airflow configuration and version.

## 12. What is execution_timeout?

It limits how long a task may execute. It protects resources from tasks that hang or run much longer than expected. The value should allow normal workload variation.

## 13. What is depends_on_past?

It can require the previous scheduled task instance to succeed before the current one runs. It is useful for workflows that truly require sequential period completion, but can block future runs after an earlier failure.

## 14. What is an Airflow pool?

A pool limits the number of concurrent task instances using a shared resource. For example, a pool can prevent too many extraction tasks from overloading a source database.

## 15. Why should a pipeline use the data interval rather than datetime.now()?

A task may execute after midnight or be backfilled for a historical date. datetime.now() describes the worker's current time, not the intended business period. The run's data interval provides the correct period context.

## 16. Why is idempotency important for retries?

A task may partially write data and then fail. If a retry appends the same records again, duplicates result. Idempotent operations make repeated execution produce the intended final state, for example by safely replacing a date partition or merging on a stable key.

## 17. Scenario: a task fails due to a network timeout.

I would inspect the error and determine whether it is transient. If it is, a limited retry policy with an appropriate delay or backoff can help. I would also monitor repeated failures and avoid retrying indefinitely.

## 18. Scenario: a task fails because a required column is missing.

This is a permanent data-contract failure until the source or mapping is corrected. Repeating the same task with the same input is unlikely to help. I would fail clearly, alert the owner and prevent downstream publication.

## 19. Scenario: a DAG was paused for 60 days and is now enabled.

I would inspect catchup configuration, estimate the number of historical intervals, review source retention and output idempotency, and plan capacity. I would not blindly release all historical runs against a production source.

## 20. Scenario: a historical calculation must be corrected.

I would define the exact backfill range, confirm source availability, deploy the corrected code, use idempotent output, limit concurrency, run a small test interval and reconcile results before completing the full backfill.

## 21. Explain your Day 62 project.

I configured a retry policy and timeout, distinguished temporary from permanent failures, designed safe reruns and documented a historical backfill plan. I also explained why the DAG's data interval determines the business date and how pools can protect shared systems.

## Common interview traps

- Treating logical date as actual task start time.
- Using current date for historical data processing.
- Enabling catchup without estimating the backlog.
- Retrying permanent data-quality failures.
- Retrying append operations without idempotency.
- Backfilling without a capacity or reconciliation plan.
- Assuming more parallelism always improves throughput.

## Readiness checklist

- [ ] Explain DAG run and task instance.
- [ ] Explain logical date and data interval.
- [ ] Explain catchup and backfill.
- [ ] Explain retry, delay, backoff and timeout.
- [ ] Explain depends_on_past and pools.
- [ ] Describe safe recovery for temporary and permanent failures.
- [ ] Design an idempotent backfill.
