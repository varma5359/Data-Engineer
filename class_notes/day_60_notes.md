# Day 60: Workflow Orchestration Fundamentals

**Phase:** 7 — Orchestration, Quality and Monitoring  
**Learning time:** 2–3 hours  
**Audience:** Complete beginners and junior Data Engineer candidates  
**Primary tool:** Apache Airflow (concepts first; hands-on DAG coding begins on Day 61)  
**Topic boundary:** Class notes only. The project workbook and interview guide are separate files.

> In earlier phases, you wrote scripts that read, transform and write data. But who starts those scripts every morning? Who makes sure step B waits for step A? Who retries a failed task and tells the team that the pipeline failed? Workflow orchestration solves these coordination problems. Today you will understand orchestration, scheduling, DAGs, task dependencies and Airflow architecture before writing your first DAG tomorrow.

## Learning outcomes

By the end of this lesson, you should be able to:
1. Explain why a data pipeline needs orchestration.
2. Distinguish a script, scheduler and orchestrator.
3. Compare scheduling with orchestration.
4. Explain a workflow, task and dependency.
5. Understand what a DAG is and why it must be acyclic.
6. Read a simple workflow diagram.
7. Explain Apache Airflow's main components.
8. Understand the difference between DAG parsing and task execution.
9. Identify metadata database, scheduler, executor, worker and web interface responsibilities.
10. Explain retries, task states, logs and manual reruns at a beginner level.
11. Decide when Airflow is suitable and when a simpler scheduler is enough.

## Today's learning flow

Business deadline -> Pipeline steps -> Dependencies -> DAG -> Scheduler -> Executor/worker -> Logs and monitoring

---

# Part 1 — Why orchestration is needed

## 1. Start with a real business problem

Imagine TechStore wants a sales report every morning at 7:00 AM.

The report needs data from three sources:
- order files;
- customer database;
- product reference file.

The work has several steps:

1. Check whether today's order file arrived.
2. Read orders.
3. validate the file schema;
4. remove invalid records;
5. load customer information;
6. join orders with customers and products;
7. calculate daily sales;
8. write the curated dataset;
9. run quality checks;
10. notify the reporting team.

A Python script can perform some or all of these operations. But the company also needs to control when the steps run, what happens if a step fails, and whether the next step is allowed to start.

That coordination is workflow orchestration.

## 2. What is workflow orchestration?

**Beginner definition:** Workflow orchestration is the process of coordinating multiple tasks so they run in the correct order, at the correct time, with defined dependencies and failure-handling rules.

Think of an orchestra. Different musicians play different parts. The conductor coordinates when each part starts and how the complete performance proceeds.

In a data pipeline:
- a task performs one unit of work;
- a dependency defines what must finish first;
- a scheduler decides when a workflow run should start;
- an executor decides how tasks are run;
- workers execute task code;
- monitoring shows whether the workflow succeeded.

## 3. What happens without orchestration?

Suppose you run four scripts manually:

~~~text
python extract.py
python clean.py
python transform.py
python report.py
~~~

Problems:
- A person must remember to start each script.
- If extract.py fails, someone might still run clean.py.
- There is no central view of the complete workflow.
- Retry behavior may be inconsistent.
- The business cannot easily see whether the 7:00 AM report is ready.
- Manual execution becomes difficult as pipelines grow.

Manual scripts are useful for learning and small jobs. They become difficult to coordinate at scale.

## 4. A simple comparison

| Without orchestration | With orchestration |
|---|---|
| Start scripts manually | Start workflows by schedule or trigger |
| Person remembers the order | Dependencies define the order |
| Each script reports separately | Central workflow status |
| Retry is manual and inconsistent | Retry policy can be configured |
| Hard to see full pipeline | Workflow graph and task history |
| Failure investigation is scattered | Task logs and run metadata in one place |

Orchestration does not automatically make bad code reliable. It coordinates the code and its operational behavior.

---

# Part 2 — Scheduling versus orchestration

## 5. What is scheduling?

Scheduling means deciding when a job should run.

Examples:
- every day at 6:00 AM;
- every hour;
- every Monday;
- when a file arrives;
- after a user manually starts it.

A scheduler answers: **When should this job start?**

## 6. What is orchestration?

Orchestration answers more questions:
- When should the workflow start?
- Which task runs first?
- Which task depends on another?
- Which tasks can run in parallel?
- What happens when a task fails?
- Should a later task be skipped?
- Where can the operator inspect logs?
- How can a failed run be recovered?

## 7. Scheduling and orchestration together

A scheduler can start a workflow. The orchestrator coordinates the workflow's tasks.

~~~text
Schedule says: Start at 6:00 AM
             |
             v
       Workflow starts
             |
             v
      Check source file
             |
             v
      Validate source
             |
             v
       Transform data
             |
             v
       Publish output
~~~

**Remember:** Scheduling is one capability within a larger orchestration process.

## 8. When is a simple scheduler enough?

If one independent script runs once every night and has no dependencies, a basic operating-system scheduler may be enough.

If a workflow has multiple dependent tasks, retries, backfills, monitoring requirements and multiple owners, a workflow orchestrator can provide more value.

Do not introduce a complex orchestrator for every small script. Choose based on operational needs.

---

# Part 3 — Understand a workflow

## 9. What is a workflow?

A workflow is a defined sequence or graph of tasks that together complete a business process.

TechStore daily sales workflow:

~~~text
Start
  |
  v
Check input files
  |
  v
Validate input
  |
  v
Transform orders
  |
  v
Load warehouse
  |
  v
Run quality checks
  |
  v
Publish report
  |
  v
End
~~~

Each box represents a task or a logical step.

## 10. What is a task?

A task is one unit of work in a workflow.

Examples:
- check whether a file exists;
- extract data from an API;
- run a SQL query;
- execute a Spark job;
- validate a row count;
- send a notification.

A task should have a clear responsibility.

A beginner mistake is to put an entire large data platform inside one task. That makes failures difficult to locate and recover.

## 11. What is a dependency?

A dependency describes the order relationship between tasks.

Example:
- transform_orders depends on validate_orders;
- validate_orders depends on check_file.

This means transform_orders must not start until validate_orders has reached a state that satisfies the dependency rule.

~~~text
check_file -> validate_orders -> transform_orders
~~~

## 12. Why dependencies matter

Suppose the transformation starts before the source file has arrived.

Possible outcomes:
- the job fails because the file does not exist;
- the job reads yesterday's file accidentally;
- the job publishes incomplete data;
- a business report becomes incorrect.

A dependency prevents a downstream task from starting before its required upstream task is complete.

## 13. Sequential tasks

Sequential tasks run in an order.

~~~text
A -> B -> C -> D
~~~

Example:
- A = extract;
- B = validate;
- C = transform;
- D = publish.

If B fails, C and D should not run under the usual success-only dependency rule.

## 14. Parallel tasks

Some tasks do not depend on each other.

For example, after validating the source file:
- one task can calculate sales by product;
- another task can calculate sales by region.

~~~text
                 +-> sales_by_product --+
validate_source -|                       |-> combine_report
                 +-> sales_by_region ---+
~~~

Parallel work can reduce elapsed time, but it uses more compute resources. The tasks must truly be independent.

## 15. A task should have one clear purpose

Good task names:
- check_orders_file;
- validate_orders_schema;
- transform_orders;
- load_sales_table;
- verify_sales_totals.

Weak task names:
- do_everything;
- process_data;
- step1;
- final_work.

A clear name helps the operator understand what failed without opening the code first.

---

# Part 4 — What is a DAG?

## 16. Expand the abbreviation

DAG means **Directed Acyclic Graph**.

Break it into three words:

| Word | Simple meaning |
|---|---|
| Directed | Connections have a direction |
| Acyclic | A path cannot loop back to an earlier node |
| Graph | A set of nodes connected by edges |

In Airflow, a DAG describes the workflow structure: its tasks, dependencies, schedule and related configuration.

## 17. Directed means an arrow has a direction

~~~text
extract -> validate -> transform
~~~

This says extract is upstream of validate, and validate is upstream of transform.

The arrow does not mean the data itself is necessarily passed through the arrow. It expresses workflow dependency. Data may be passed using a database, object storage or a supported task communication mechanism.

## 18. Acyclic means no dependency loop

Valid:

~~~text
A -> B -> C
~~~

Invalid:

~~~text
A -> B -> C
^         |
|_________|
~~~

The invalid graph says A must wait for C while C waits for B and B waits for A. No task can make progress.

Airflow DAG dependencies must be acyclic.

## 19. DAG versus data flow

A DAG defines task order. It does not automatically define how large datasets move between tasks.

For example:
- Task A writes a Parquet file to object storage.
- Task B reads that file.
- The DAG dependency ensures B starts after A succeeds.
- The file path or table is the data handoff.

Avoid passing a huge Pandas DataFrame through a small orchestration metadata channel. Store the data in a suitable data system and pass a reference.

## 20. DAG versus task

| DAG | Task |
|---|---|
| Describes the whole workflow | Performs one unit of work |
| Contains multiple tasks | Belongs to a workflow |
| Defines schedule and dependencies | Has execution logic and task state |
| Example: daily_sales_pipeline | Example: validate_orders |

---

# Part 5 — Apache Airflow introduction

## 21. What is Apache Airflow?

Apache Airflow is an open-source platform for authoring, scheduling and monitoring workflows programmatically.

You define workflows as Python code. Airflow reads the DAG definitions, schedules workflow runs and coordinates task execution through its configured components.

Airflow is widely used for batch-oriented data workflows.

## 22. What Airflow is good at

- scheduling Python and command-line work;
- coordinating SQL and data processing jobs;
- expressing task dependencies;
- retrying failed tasks;
- tracking workflow and task status;
- viewing logs;
- rerunning tasks and performing backfills;
- integrating with external systems through operators and providers.

## 23. What Airflow is not

Airflow is not:
- a database for storing all business data;
- a replacement for Spark;
- a replacement for a warehouse;
- automatically a real-time event streaming engine;
- a guarantee that task code is correct;
- a substitute for idempotent data processing.

A useful separation:

~~~text
Airflow = decides and coordinates when/how tasks run
Spark   = processes distributed data
Database = stores structured records
S3      = stores objects and files
Kafka   = transports and retains event streams
~~~

Airflow can trigger these systems. It does not replace them.

---

# Part 6 — Airflow architecture

## 24. Why does Airflow have multiple components?

A workflow platform needs to:
- read workflow definitions;
- decide when a run is due;
- track task state;
- send tasks to an execution system;
- run the task code;
- store logs and metadata;
- show status to operators.

Airflow separates these responsibilities.

## 25. Main architecture components

| Component | Beginner explanation |
|---|---|
| DAG files | Python files that define workflows |
| DAG processor | Parses DAG files and makes definitions available |
| Scheduler | Creates/schedules runs and decides which tasks are ready |
| Executor | Coordinates how task instances are executed |
| Worker | A process or machine that runs task code in worker-based setups |
| Metadata database | Stores Airflow's operational metadata and task/run states |
| Web interface | Lets users inspect DAGs, runs, tasks and logs |
| Triggerer | Handles deferrable tasks that wait asynchronously |

Exact process boundaries and deployment topology depend on the Airflow version and executor. Some local executors run task processes on the same host; distributed executors use separate workers.

## 26. DAG files

A DAG file is Python code that defines:
- a DAG;
- tasks;
- task dependencies;
- scheduling and configuration.

The DAG file is not simply a script that Airflow runs from top to bottom as one job.

Airflow parses the file to discover the workflow structure. Actual task execution happens later through task instances.

## 27. DAG processor

The DAG processor reads and parses DAG files.

It needs the files to be valid Python and their imports to be available in the Airflow environment.

A syntax error or unavailable import can prevent a DAG from being loaded.

## 28. Scheduler

The scheduler:
- checks which DAG runs are due;
- evaluates task dependencies;
- determines which task instances are eligible to run;
- submits eligible work through the configured executor.

The scheduler coordinates work. It is not necessarily the process that executes every task's business code.

## 29. Executor

The executor is responsible for the task execution strategy.

Common executor types include:
- SequentialExecutor;
- LocalExecutor;
- CeleryExecutor;
- KubernetesExecutor.

Availability and recommended use depend on Airflow version and deployment configuration.

Beginner view:
- Sequential: tasks execute in a simple sequential execution model.
- Local: task processes run locally.
- Celery: tasks are distributed to worker processes through a queue.
- Kubernetes: tasks run in Kubernetes-managed pods.

Do not select an executor only because its name sounds more powerful. Consider workload, infrastructure, operations and team capability.

## 30. Worker

A worker executes task code in a worker-based architecture.

For example, the worker may:
- run a Python function;
- execute a shell command;
- submit a Spark job;
- run a SQL operation.

The worker needs the appropriate Python packages, credentials, network access and configuration.

A DAG parsing environment and a task execution environment may differ in distributed deployments.

## 31. Metadata database

The metadata database stores Airflow's operational records, such as:
- DAG run records;
- task instance states;
- scheduling information;
- connections and variables metadata;
- configuration-related records.

It is not the warehouse for customer orders or sales facts.

Keep application/business data in the appropriate database, lake or warehouse.

## 32. Web interface

The web interface helps users:
- view DAGs;
- inspect workflow runs;
- inspect task states;
- open task logs;
- trigger a DAG manually;
- clear or rerun tasks when permitted;
- review duration and history.

A green status does not prove that the business data is correct. Data quality checks are still required.

## 33. Triggerer

Some operators can defer while waiting for an external condition rather than occupying a worker slot continuously.

The triggerer handles the asynchronous waiting part for deferrable tasks.

You do not need to use deferrable operators in every beginner DAG.

---

# Part 7 — How an Airflow workflow runs

## 34. The high-level lifecycle

~~~text
Python DAG file
      |
      v
DAG processor parses the definition
      |
      v
Scheduler evaluates schedule and dependencies
      |
      v
Eligible task instance is submitted
      |
      v
Executor arranges execution
      |
      v
Worker/process runs task code
      |
      v
Task state and logs are recorded
      |
      v
Web interface displays status
~~~

## 35. Task states

Common task states include:

| State | Meaning |
|---|---|
| None / no state | Task has not started in the run |
| Scheduled | Ready for scheduling |
| Queued | Sent for execution |
| Running | Task code is executing |
| Success | Task completed successfully |
| Failed | Task execution failed |
| Up for retry | Failed attempt will be retried |
| Skipped | Task was intentionally skipped |
| Upstream failed | Required upstream task did not succeed |
| Deferred | Task is waiting using a deferrable mechanism |

Exact state names and transitions may vary by Airflow version.

## 36. What happens if a task fails?

Example:

~~~text
check_file -> validate_file -> transform -> publish
                  |
                FAILED
~~~

Under a normal all-success dependency rule:
- transform does not run;
- publish does not run;
- the failed task records a failure;
- a configured retry may be attempted;
- the operator can inspect logs and decide what to do.

The downstream tasks are protected from consuming output that was not validated.

## 37. Why task logs matter

Logs help answer:
- Which task ran?
- What input did it process?
- Which step failed?
- What exception occurred?
- How long did the task run?
- Did it connect to the correct system?

Logs must not expose passwords, access tokens, personal data or other secrets.

---

# Part 8 — Orchestration design principles

## 38. Keep tasks focused

Prefer several understandable tasks over one enormous task.

However, do not create hundreds of tiny tasks without a reason. Every task adds scheduling and operational overhead.

Choose boundaries that represent meaningful steps, failure boundaries or reusable operations.

## 39. Make tasks repeatable

A task may be retried or rerun.

For example, a task that appends yesterday's sales every time it runs can create duplicates.

Design the data operation to be idempotent:
- replace the intended date partition safely;
- merge by a stable business key;
- use a transaction or supported table commit;
- record the run and source identifiers.

The correct approach depends on the destination.

## 40. Do not put large data in orchestration messages

Airflow task communication such as XCom is intended for small metadata values, not large datasets.

Good value to pass:
- output file path;
- batch ID;
- row count;
- status string.

Poor value to pass:
- 2 GB DataFrame;
- millions of order rows;
- entire Parquet file contents.

Store large data in S3, a database, a warehouse or another suitable data platform. Pass the reference.

## 41. Keep secrets out of DAG code

Never write credentials directly into a DAG file.

Use approved Airflow Connections, a secrets backend, workload identity or another secure credential mechanism.

Day 65 covers this in detail.

## 42. Separate workflow definition from business logic

A useful structure:

~~~text
project/
    dags/
        daily_sales_dag.py
    src/
        extract_orders.py
        transform_orders.py
        validate_orders.py
    tests/
    requirements.txt
~~~

The DAG coordinates the functions or commands. The business logic can be tested independently.

For small classroom examples, a short function inside a DAG is acceptable. As the project grows, separate modules improve maintainability.

---

# Part 9 — A workflow design exercise

## 43. Requirements

TechStore requires a daily report.

Rules:
- source file must exist;
- schema must be valid;
- invalid records must not be published;
- transformation must finish before loading;
- quality checks must pass before report publication.

## 44. Identify the tasks

| Task ID | Responsibility |
|---|---|
| check_source | Confirm required file arrived |
| validate_source | Check schema and required fields |
| transform_orders | Apply business rules |
| load_warehouse | Load curated records |
| quality_check | Reconcile row counts and totals |
| publish_report | Make report available |

## 45. Draw the DAG

~~~text
check_source
     |
     v
validate_source
     |
     v
transform_orders
     |
     v
load_warehouse
     |
     v
quality_check
     |
     v
publish_report
~~~

## 46. Think through a failure

If validate_source fails:
- transform_orders must not run;
- load_warehouse must not run;
- quality_check must not run;
- publish_report must not run;
- the operator inspects the validation log and source file.

If publish_report fails after the warehouse load succeeded, the recovery plan should rerun publication without duplicating the warehouse load. This requires task boundaries and idempotent behavior.

---

# Part 10 — Common beginner mistakes

| Mistake | Why it causes trouble | Better approach |
|---|---|---|
| Confusing scheduler with orchestrator | Scheduling alone does not coordinate all task behavior | Learn schedule, dependency, execution and recovery separately |
| Putting all logic in one task | Difficult to identify and recover failures | Use meaningful task boundaries |
| Creating circular dependencies | A DAG cannot execute a cycle | Draw and validate an acyclic graph |
| Treating Airflow as a data warehouse | Metadata storage is not analytical storage | Keep business data in the right platform |
| Passing large data through XCom | Can overload metadata storage | Pass paths, IDs and small metadata |
| Hardcoding credentials | Exposes secrets | Use connections or secrets management |
| Assuming a green DAG means correct data | Task success is not data quality | Add explicit validation tasks |
| Retrying non-idempotent writes blindly | May duplicate data | Design safe reruns |
| Installing packages only on one worker | Other execution nodes may fail | Keep runtime dependencies consistent |
| Writing expensive code at DAG parse time | Slows parsing and may cause scheduler issues | Keep DAG definition lightweight |

---

# Part 11 — Knowledge check

## 47. Questions with answers

**Q1. What is workflow orchestration?**  
It coordinates tasks, dependencies, schedules, execution and failure handling across a workflow.

**Q2. What is the difference between scheduling and orchestration?**  
Scheduling decides when a job starts. Orchestration coordinates the complete workflow, including task order, dependencies, retries and monitoring.

**Q3. What does DAG stand for?**  
Directed Acyclic Graph.

**Q4. Why must a DAG be acyclic?**  
A cycle creates a dependency loop in which tasks can wait on one another and cannot progress.

**Q5. Does a DAG arrow transfer a DataFrame?**  
No. It defines a dependency. Data is normally stored in a database, object store or another appropriate data system.

**Q6. What does the Airflow scheduler do?**  
It evaluates schedules and dependencies and determines which task instances are ready to run.

**Q7. What does an executor do?**  
It coordinates the configured method of running task instances.

**Q8. What is the metadata database used for?**  
It stores Airflow operational metadata such as DAG runs and task states, not the company's main business datasets.

**Q9. Is Airflow a streaming engine?**  
Airflow is primarily a workflow orchestration platform, commonly used for scheduled and batch-oriented workflows. It can coordinate streaming jobs but does not replace the streaming engine.

**Q10. What should tasks pass between one another?**  
Small metadata such as identifiers, paths and counts. Large datasets should remain in suitable storage or processing systems.

---

# Part 12 — Revision sheet

| Term | Remember |
|---|---|
| Workflow | Connected business process made of tasks |
| Task | One unit of work |
| Dependency | Defines required task order |
| Scheduler | Decides when work is due |
| Orchestrator | Coordinates workflow execution and operations |
| DAG | Directed Acyclic Graph |
| Directed | Connections have direction |
| Acyclic | No dependency loop |
| DAG processor | Reads and parses DAG definitions |
| Scheduler | Determines due runs and ready tasks |
| Executor | Coordinates task execution strategy |
| Worker | Runs task code in worker-based setups |
| Metadata DB | Stores orchestration state and metadata |
| Web interface | Shows workflow status and logs |
| Triggerer | Handles asynchronous waiting for deferrable tasks |
| Retry | Another attempt after failure |
| XCom | Small task-to-task metadata communication |
| Idempotency | Safe repeat execution with the intended final result |

## Final self-check

- [ ] Explain why pipelines need orchestration.
- [ ] Distinguish scheduling and orchestration.
- [ ] Explain tasks and dependencies.
- [ ] Draw a valid DAG.
- [ ] Identify a circular dependency.
- [ ] Explain Airflow's architecture.
- [ ] Distinguish DAG parsing from task execution.
- [ ] Explain common task states.
- [ ] Explain why large datasets should not be passed through XCom.
- [ ] Explain why idempotency matters.
- [ ] Explain why Airflow does not replace Spark, Kafka, S3 or a warehouse.

**Next lesson — Day 61: Airflow DAGs and Tasks.** You will create your first practical DAG, define tasks, use operators and dependencies, run a Python-based workflow and inspect task output.
