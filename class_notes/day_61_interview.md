# Day 61 Interview Guide — Airflow DAGs and Tasks

## 1. What is an Airflow DAG file?

It is a Python file that defines a workflow, its tasks, dependencies, schedule and configuration. Airflow parses the file to discover the DAG. Task code executes later when a DAG run schedules the task instance.

## 2. What is dag_id?

dag_id is the unique identifier for a DAG within an Airflow environment. It appears in the UI and CLI and is used to reference the workflow.

## 3. What does the @dag decorator do?

It marks a Python function as a DAG factory. Calling the decorated function registers the DAG definition with the configured settings and tasks.

## 4. What does the @task decorator do?

It converts a Python function into an Airflow task. Calling the task inside the DAG definition creates a task invocation and can create dependencies through its returned output.

## 5. What is the TaskFlow API?

TaskFlow is an Airflow programming style that uses Python decorators such as @dag and @task to define workflows and tasks. It can make Python task dependencies and small return values easier to express.

## 6. What is PythonOperator?

PythonOperator is an Airflow operator that executes a supplied Python callable. It is useful when you want to configure a Python task explicitly using the operator interface.

## 7. What is the difference between TaskFlow and PythonOperator?

TaskFlow uses decorators and function calls to define tasks and dependencies. PythonOperator accepts a Python callable and operator parameters. Both can execute Python code; the choice depends on project style and integration needs.

## 8. What is schedule=None?

It means the DAG has no time-based automatic schedule in that example. It can still be triggered manually or through another supported trigger mechanism.

## 9. What does catchup=False mean?

It prevents Airflow from automatically creating all missed historical scheduled runs between the configured start date and the current scheduling period. Backfill behavior should be considered separately.

## 10. Why is start_date important?

It establishes the earliest scheduling boundary for a DAG. Airflow scheduling is based on data intervals and logical dates, so start_date should be configured carefully with a timezone. It is not simply a promise that the task starts at that exact wall-clock moment.

## 11. Why should DAG parsing be lightweight?

Airflow repeatedly parses DAG files to discover definitions. External API calls, large file reads or database queries at module import time can slow parsing and create instability. Put actual work inside tasks.

## 12. Why pass a file path rather than a DataFrame through task metadata?

Task metadata channels such as XCom are intended for small values. A file path is small and points to data stored in a suitable system. Passing a huge DataFrame can overload metadata storage and make workflows fragile.

## 13. How do you inspect DAG import errors?

Use the Airflow CLI command for listing DAG import errors and inspect scheduler/DAG processor logs. Check syntax, missing imports, incompatible providers and packages in the parsing environment.

## 14. How do you trigger a manual DAG run?

In a permitted environment, use the Airflow UI or the supported CLI command, such as airflow dags trigger followed by the DAG ID. Do not trigger a shared production workflow without authorization.

## 15. What is the difference between a DAG run and a task instance?

A DAG run is one execution of a workflow for a particular run/data interval or manual trigger. A task instance is the execution of one task within that DAG run.

## 16. Scenario: the DAG is not visible in the UI.

I would check the configured DAG folder, file permissions, DAG processor status and import errors. I would also verify that the DAG factory is invoked and the Python file is compatible with the installed Airflow version.

## 17. Scenario: a task is visible but fails immediately.

I would open the task log, identify the exception and check imports, environment variables, permissions, dependencies and network access. I would not assume the scheduler itself is broken.

## 18. Scenario: a task works locally but fails on a worker.

The worker may have a different Python environment, package version, file path, identity or network access. I would compare the execution environments and ensure dependencies and configuration are deployed consistently.

## 19. Why test business logic separately from the DAG?

Pure business functions can be tested quickly without starting Airflow. This separates calculation defects from orchestration or environment defects. A standalone unit test does not prove that the DAG parses or runs correctly.

## 20. Explain your Day 61 project.

I created a daily ETL DAG with source checking, validation, transformation and publication tasks. I used TaskFlow decorators, passed small metadata values between tasks and inspected DAG discovery, task states and logs. I introduced a controlled validation failure and confirmed downstream tasks were blocked. I documented which checks were standalone Python tests and which were performed in Airflow.

## Common interview traps

- Confusing DAG parsing with task execution.
- Assuming schedule=None disables manual runs.
- Treating start_date as an exact task start time.
- Passing large datasets through XCom.
- Putting database/API calls at module import time.
- Claiming a Python syntax check proves Airflow runtime success.
- Ignoring worker environment differences.

## Readiness checklist

- [ ] Explain DAG ID and DAG run.
- [ ] Explain @dag and @task.
- [ ] Explain PythonOperator.
- [ ] Explain schedule, start_date and catchup.
- [ ] Explain task dependencies and XCom.
- [ ] Explain DAG parsing errors.
- [ ] Explain how to inspect logs and worker failures.
