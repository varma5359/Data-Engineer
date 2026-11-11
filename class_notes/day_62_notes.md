# Day 62: Airflow Scheduling and Reliability

**Phase:** 7 — Orchestration, Quality and Monitoring  
**Learning time:** 2–3 hours  
**Audience:** Complete beginners and junior Data Engineer candidates  
**Tool:** Apache Airflow 2.x concepts and examples  
**Topic boundary:** Class notes only. Project workbook and interview guide are separate.

> A DAG can be written correctly and still run at the wrong time, process the wrong date or create duplicate data after a retry. Today you will understand how Airflow schedules DAG runs, what logical dates and data intervals mean, how catchup and backfills work, and how retries and recovery should be designed.

## Learning outcomes

- Explain a DAG run and task instance.
- Understand schedule intervals, logical dates and data intervals.
- Distinguish execution time from the business data period.
- Explain catchup and backfill.
- Configure retry-related parameters.
- Understand retry delay and exponential backoff.
- Explain task failure and downstream behavior.
- Design idempotent reruns.
- Understand pools and concurrency at a high level.
- Identify risks in backfills and historical reprocessing.

## 1. Start with a business deadline

TechStore needs yesterday's sales report by 7:00 AM every day.

At 7:00 AM on November 11, the report should contain orders from November 10.

This creates two different time ideas:
- when the workflow runs;
- which business period the workflow processes.

Do not confuse them.

## 2. What is a DAG run?

A DAG run is one execution of a DAG.

For a daily sales workflow, there may be one run for each daily data interval. A manual trigger may create a separate run with its own run identity and configuration.

A DAG run contains task instances for the tasks in that workflow.

## 3. What is a task instance?

A task instance is one task's execution within one DAG run.

For example:
- DAG: daily_sales_pipeline;
- DAG run: daily interval for 2026-11-10;
- task: validate_orders;
- task instance: validate_orders for that particular DAG run.

The same task can have many task instances across different runs.

## 4. Logical date and data interval

Airflow uses logical dates and data intervals to represent scheduled periods.

For a daily workflow, a run associated with a data interval from November 10 00:00 to November 11 00:00 commonly processes the data for November 10.

The scheduled run is generally created after the interval has ended, not at the beginning of the interval.

The logical date identifies the run's scheduling period; it is not necessarily the real wall-clock time when the task starts.

## 5. Simple timeline

~~~text
Nov 10, 00:00                         Nov 11, 00:00
      |------------------------------------|
              Data interval for Nov 10

After the interval closes:
Airflow can schedule the corresponding daily run.
Tasks execute when capacity and dependencies allow.
~~~

The exact timetable and displayed logical date depend on the DAG's schedule and timezone.

## 6. Why this matters

Suppose a task uses the current system date instead of the DAG's data interval.

A run that is processing November 10 data might accidentally read November 11 files because the task executes after midnight.

For reliable pipelines, derive the processing period from the DAG run's data interval or an explicitly supplied business date. Do not blindly use datetime.now() to decide which partition to process.

## 7. Understand schedule examples

Common schedule expressions include:

| Schedule | Meaning |
|---|---|
| @hourly | Hourly schedule |
| @daily | Daily schedule |
| @weekly | Weekly schedule |
| 0 6 * * * | Cron expression: daily at 06:00 in the configured timezone |
| None | No time-based automatic schedule |

Cron expressions have five fields:
minute hour day-of-month month day-of-week.

Example:

~~~text
0 6 * * *
| | | | |
| | | | +-- day of week
| | | +---- month
| | +------ day of month
| +-------- hour
+---------- minute
~~~

Timezone matters. A 6:00 AM schedule in UTC is not necessarily 6:00 AM in India.

## 8. Use timezone-aware start dates

Example:

~~~python
import pendulum

start_date = pendulum.datetime(
    2026, 1, 1,
    tz="Asia/Kolkata"
)

print(start_date)
~~~

Expected output shape:

~~~text
2026-01-01T00:00:00+05:30
~~~

A timezone-aware date makes the intended local scheduling boundary explicit.

## 9. What is catchup?

catchup controls whether Airflow creates runs for historical schedule intervals that were missed after the DAG's start date.

Example:
- DAG start date: January 1;
- DAG deployed: January 5;
- schedule: daily.

With catchup enabled, Airflow may create runs for missed intervals from January 1 onward, subject to the configured timetable and limits.

With catchup disabled, Airflow does not automatically create all those missed historical runs.

## 10. Why can catchup be dangerous?

Imagine a DAG was paused for 90 days. When it is enabled, catchup may schedule many historical runs.

This can:
- create a large compute load;
- overload a source database;
- generate unexpected cloud costs;
- write historical data repeatedly;
- trigger rate limits.

Always understand the intended historical processing policy before enabling catchup.

## 11. What is a backfill?

A backfill is the deliberate creation and execution of workflow runs for historical data intervals.

Example:
The company discovers that the sales calculation was wrong for November 1 through November 5. The team needs to reprocess those dates.

A backfill should have:
- approved date range;
- corrected code or configuration;
- source availability;
- idempotent output strategy;
- capacity and cost review;
- validation and reconciliation;
- clear ownership.

Backfill commands and interfaces vary by Airflow version. Follow the version-specific official CLI documentation.

## 12. Catchup versus backfill

| Catchup | Backfill |
|---|---|
| Scheduling behavior for missed intervals | Deliberate historical reprocessing |
| Controlled by DAG scheduling configuration | Initiated as an operational action |
| Can create multiple historical runs | Targets a selected historical range |
| Must be planned to avoid load | Must be planned to avoid load |

## 13. What is a retry?

A retry is another attempt to execute a task after a failure.

Example:
A database connection times out for a few seconds. A retry may succeed after the database becomes available.

Retries are useful for temporary failures. They do not fix invalid code or bad input.

## 14. Configure retries

Example using the TaskFlow API:

~~~python
import pendulum

from airflow.decorators import dag, task


@dag(
    dag_id="day62_retry_example",
    start_date=pendulum.datetime(2026, 1, 1, tz="UTC"),
    schedule=None,
    catchup=False,
    default_args={
        "retries": 2,
    }
)
def retry_example():

    @task
    def check_connection():
        print("Checking source connection")

    check_connection()


retry_example()
~~~

Here retries=2 means Airflow can make two retry attempts after the initial attempt, subject to task state and configuration.

## 15. Retry delay

retry_delay controls the waiting time before a retry.

Example:

~~~python
from datetime import timedelta

default_args = {
    "retries": 3,
    "retry_delay": timedelta(minutes=5)
}
~~~

This example requests a five-minute delay between retry attempts.

## 16. Exponential backoff

Exponential backoff increases the wait between repeated attempts.

Example concept:
- attempt 1 fails -> wait about 1 minute;
- attempt 2 fails -> wait about 2 minutes;
- attempt 3 fails -> wait about 4 minutes.

Actual delay calculation and caps depend on Airflow settings and version.

In Airflow task configuration, retry_exponential_backoff can enable exponential backoff for supported task retry behavior.

## 17. When should a task be retried?

Usually retry temporary failures:
- network timeout;
- temporary service unavailability;
- rate limiting, with an appropriate wait;
- short-lived database connection issue.

Usually do not blindly retry permanent failures:
- missing required column;
- invalid date format;
- permission denied caused by wrong IAM policy;
- Python syntax error;
- business rule violation.

A retry policy should match the failure type.

## 18. What is a timeout?

A task execution timeout limits how long a task is allowed to run.

Example:

~~~python
from datetime import timedelta

execution_timeout = timedelta(minutes=30)
~~~

A timeout helps prevent a stuck task from consuming resources indefinitely. Choose it based on normal workload duration and acceptable variation.

## 19. What is depends_on_past?

This setting can require a task's previous scheduled instance to succeed before the current instance runs.

It can be useful for sequential period processing, but it can also block future runs if one earlier run fails.

Use it only when the business process truly requires prior-period success. It is not a general solution for all pipelines.

## 20. What is a pool?

A pool limits how many task instances can use a shared resource at the same time.

Example:
A source database can safely handle only three heavy extraction jobs. An Airflow pool can limit concurrent tasks that use that database.

Pools protect shared systems and help manage capacity.

## 21. Concurrency and parallelism

Several settings can influence how many DAG runs or task instances execute at once. Exact configuration names and defaults vary by Airflow version.

The design questions are:
- How many runs can overlap?
- How many tasks can run at once?
- How many workers are available?
- What can the database/source system handle?
- Does the output support concurrent writes?

More parallelism is not always better.

## 22. Failure and recovery lifecycle

~~~text
Task starts
   |
   v
Task fails
   |
   v
Retry attempts remaining?
   | Yes
   v
Wait for retry delay -> Run again
   |
   No
   v
Mark failed -> Alert operator
   |
   v
Investigate -> Correct issue
   |
   v
Rerun safe task/run
~~~

The exact state transition is managed by Airflow, but the engineering team must design the recovery behavior.

## 23. Why idempotency is essential

Suppose a load task appends 10,000 sales rows and then fails before recording success.

A retry may append the same 10,000 rows again.

Safer patterns:
- replace a well-defined date partition;
- merge/upsert using a stable order ID;
- use a transaction or table format with commit support;
- record source batch IDs and completed runs.

Never use broad overwrite on a production dataset without confirming its scope.

## 24. Use the DAG run's data interval

TaskFlow tasks can receive Airflow context values in supported ways. The exact access pattern depends on Airflow version and task style.

Conceptually, the task should use the run's data interval start and end to select the business period.

Do not use the worker's current date as a substitute for the intended data interval.

## 25. Backfill safety checklist

Before reprocessing historical intervals:
1. Confirm the exact date range.
2. Confirm source files/tables still exist.
3. Confirm code version and business rules.
4. Check whether output is idempotent.
5. Estimate number of runs and compute cost.
6. Protect source systems with pools/concurrency limits.
7. Run a small test interval.
8. Reconcile row counts and totals.
9. Communicate the impact to downstream users.
10. Record the backfill result.

## 26. Common scheduling mistakes

| Mistake | Why it is wrong | Better practice |
|---|---|---|
| Use current date for a historical run | Reprocesses the wrong partition | Use the data interval |
| Assume 6 AM UTC is 6 AM India time | Timezone mismatch | Set explicit timezone |
| Enable catchup without review | Can create many runs | Estimate backlog and capacity |
| Retry invalid input forever | Retries cannot fix bad data | Fail clearly and alert |
| Retry append writes | Can create duplicates | Make task idempotent |
| Backfill without a date plan | Can overload systems | Approve range and capacity |
| Set unlimited parallelism | Can overwhelm source systems | Use pools and limits |
| Use depends_on_past everywhere | One failure can block future periods | Use only when required |

## 27. Knowledge check

**Q1. What is a DAG run?**  
One execution of a DAG.

**Q2. What is a task instance?**  
One task's execution within one DAG run.

**Q3. What is a data interval?**  
The period of business data associated with a scheduled DAG run.

**Q4. Does a daily DAG necessarily execute at the beginning of its data interval?**  
No. The run is generally scheduled after the interval closes, and tasks execute when the scheduler and executor can run them.

**Q5. What is catchup?**  
Behavior that determines whether missed historical scheduled intervals are automatically created as DAG runs.

**Q6. What is a backfill?**  
A deliberate reprocessing of historical data intervals.

**Q7. What is a retry?**  
Another attempt after a task failure.

**Q8. Why is idempotency important?**  
A retry or rerun should not create duplicate or incorrect final data.

**Q9. What is an Airflow pool?**  
A concurrency control that limits tasks using a shared resource.

**Q10. Why should the data interval be used instead of the current date?**  
Because the run may execute later or be backfilled and must still process the intended historical period.

## Final self-check

- [ ] Explain DAG run and task instance.
- [ ] Explain logical date and data interval.
- [ ] Interpret @daily and a cron schedule.
- [ ] Explain timezone-aware scheduling.
- [ ] Explain catchup and backfill.
- [ ] Configure retries and retry delay.
- [ ] Explain exponential backoff and timeout.
- [ ] Explain depends_on_past and pools.
- [ ] Design idempotent reruns.
- [ ] Explain safe historical reprocessing.

**Next lesson — Day 63: Automated Data Quality.** You will turn business rules into executable validation tasks and prevent invalid data from reaching the curated layer.
