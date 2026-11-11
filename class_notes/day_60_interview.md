# Day 60 Interview Guide — Workflow Orchestration Fundamentals

## 1. What is workflow orchestration?

Workflow orchestration coordinates multiple tasks so they run at the correct time and in the correct order. It manages dependencies, execution, retries, status and operational visibility.

**Example:** A sales workflow checks a file, validates it, transforms orders, loads a warehouse, verifies totals and publishes a report.

## 2. What is the difference between scheduling and orchestration?

Scheduling decides when a job should start. Orchestration coordinates the wider workflow: task dependencies, parallel work, retries, failure handling and monitoring. Scheduling is one capability used by an orchestrator.

## 3. Why do data engineers use Apache Airflow?

Airflow lets engineers define workflows as code, schedule DAG runs, express dependencies, monitor task states, inspect logs and recover failed work. It is useful for coordinating batch data workflows and external systems.

## 4. What is a DAG?

A DAG is a Directed Acyclic Graph. Directed means connections have direction. Acyclic means dependencies cannot form a loop. Graph means tasks are represented as nodes connected by dependency edges.

## 5. Why must a DAG be acyclic?

A cycle can create a dependency deadlock. If task A waits for B, B waits for C and C waits for A, none can become ready. Airflow workflows must have an acyclic dependency structure.

## 6. What is a task?

A task is one unit of work within a workflow, such as checking a file, running a SQL query, transforming data or validating a result. Tasks should have clear responsibilities and useful names.

## 7. What is a task dependency?

A dependency defines which task must complete before another task can run. For example, transform_orders depends on validate_orders so transformation cannot start before validation succeeds under the normal success rule.

## 8. Does a DAG dependency transfer the actual data?

No. It defines execution order. Data should normally be stored in a suitable system such as object storage, a database or a warehouse. Tasks can pass small references such as a file path or batch ID.

## 9. What is the difference between a DAG and a task?

A DAG describes the whole workflow, including tasks, dependencies and schedule. A task is one unit of work inside that DAG.

## 10. What does the Airflow scheduler do?

The scheduler evaluates DAG schedules and task dependencies, creates or manages due DAG runs and identifies task instances that are ready for execution. It submits eligible work through the configured executor.

## 11. What is an executor?

An executor is the Airflow component that coordinates how task instances are run. Examples include Sequential, Local, Celery and Kubernetes executor models. The choice depends on workload and deployment architecture.

## 12. What is a worker?

A worker executes task code in a worker-based deployment. It needs the correct dependencies, configuration, credentials and network access to perform the task.

## 13. What is the metadata database?

It stores Airflow's operational metadata, such as DAG runs and task instance states. It is not intended to store the company's large business datasets or replace the analytical warehouse.

## 14. What does the web interface provide?

It provides a visual way to inspect DAGs, workflow runs, task states, durations and logs. Operators can also trigger workflows or rerun tasks when their permissions allow.

## 15. What is the DAG processor?

It parses DAG files and discovers workflow definitions. Syntax errors, missing imports or expensive top-level code can prevent a DAG from being parsed correctly.

## 16. What is the triggerer?

The triggerer handles asynchronous waiting for deferrable tasks. This allows supported tasks to wait for external events without continuously occupying a worker slot.

## 17. Is Airflow a replacement for Spark?

No. Airflow coordinates when and how a Spark job is submitted. Spark performs distributed data processing. They solve different problems and are commonly used together.

## 18. Is Airflow a streaming platform?

Airflow is primarily a workflow orchestrator and is commonly used for scheduled and batch workflows. It can coordinate streaming applications, but Kafka or Spark Structured Streaming performs the streaming work.

## 19. What is XCom and what should it contain?

XCom is Airflow's mechanism for sharing small values between tasks. Suitable values include a batch ID, file path, row count or status. Large DataFrames and file contents should remain in external storage.

## 20. Why should a task be idempotent?

Tasks can be retried or rerun. Idempotent behavior ensures repeating the task has the intended final effect without creating duplicate data. A controlled partition replacement or merge by a stable key can help.

## 21. Scenario: the validation task fails. Should transformation run?

Normally no. The dependency should prevent transformation from running until validation succeeds. The operator should inspect the validation log and correct or quarantine the input before retrying.

## 22. Scenario: a task succeeds but the report contains incorrect totals. Did orchestration work?

The task may have executed successfully, but technical execution success is not the same as business correctness. The workflow needs explicit data quality and reconciliation tasks that validate totals and other business rules before publication.

## 23. Scenario: one workflow contains a 90-minute task that does extraction, transformation, loading and publication.

I would evaluate whether it should be split into meaningful task boundaries. Separate steps improve failure isolation, visibility and recovery. I would avoid splitting into tiny tasks that add unnecessary scheduling overhead.

## 24. Explain your Day 60 workflow design.

I designed a DAG for a daily sales report with source checking, validation, transformation, warehouse loading, quality checks and publication. I defined dependencies, failure behavior, retry safety and data handoffs. I also mapped the workflow to Airflow components and explained that Airflow coordinates work while Spark, databases and object storage perform their respective roles.

## Common interview traps

- Saying scheduling and orchestration are identical.
- Calling a DAG a Python script that simply runs top to bottom.
- Assuming dependency arrows transport the dataset.
- Claiming Airflow replaces Spark or Kafka.
- Passing large datasets through XCom.
- Treating task success as proof of data correctness.
- Retrying a non-idempotent write without a recovery plan.

## Readiness checklist

- [ ] Define orchestration and scheduling.
- [ ] Explain DAG, task and dependency.
- [ ] Draw a valid acyclic workflow.
- [ ] Explain scheduler, executor, worker and metadata database.
- [ ] Explain DAG parsing and task execution.
- [ ] Describe XCom's appropriate use.
- [ ] Explain task failure and idempotent recovery.
