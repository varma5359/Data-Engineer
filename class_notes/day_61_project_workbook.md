# Day 61 Project Workbook — Daily ETL DAG

## Project goal

Create a beginner Airflow DAG with clear tasks and dependencies for a small daily sales workflow.

## Environment note

The class examples use the Airflow 2.x TaskFlow API. Check your installed Airflow and Python versions and use the official compatible installation instructions. Do not install or trigger workflows in a shared production environment without permission.

## Business scenario

TechStore receives an order file and wants a daily summary. The workflow must check the source, validate it, transform the data and publish a small summary identifier.

## Step 1 — Create the project structure

~~~text
airflow_project/
    dags/
        daily_etl_dag.py
    src/
        sales_logic.py
    README.md
~~~

Configure the Airflow DAG folder to point to dags/.

## Step 2 — Write a standalone business function

In src/sales_logic.py, create a small function that calculates a total from a list of amounts. Test it with [100, 200, 50]. Expected result: 350.

## Step 3 — Create the DAG

Create tasks:
1. check_source;
2. validate_source;
3. transform_orders;
4. publish_summary.

Use TaskFlow decorators. Give the DAG and each task a meaningful ID.

## Step 4 — Define dependencies

Connect the tasks in the correct order. Pass a file name, validation result or output identifier as small metadata. Do not pass a DataFrame or file contents through task metadata.

## Step 5 — Test syntax and discovery

Run Python syntax compilation. Then use Airflow CLI commands to:
- list DAGs;
- inspect import errors;
- list tasks.

Record the actual output. If Airflow is not installed, complete the design and standalone Python test, and clearly record that Airflow runtime validation remains pending.

## Step 6 — Run the workflow

In a permitted local Airflow environment:
- trigger a manual run;
- open the DAG run;
- inspect each task state;
- open task logs;
- confirm the tasks ran in the expected order.

## Step 7 — Introduce a controlled failure

Temporarily make validation return False or raise a ValueError. Run again.

Confirm that downstream transformation/publication does not proceed under the normal dependency rule. Restore the successful version afterward.

## Step 8 — Document recovery

Explain how the operator would:
- inspect the failed task log;
- correct the input or code;
- rerun the appropriate task or DAG;
- prevent duplicate output.

## Step 9 — Submission

Submit:
1. project folder;
2. DAG source file;
3. standalone business logic;
4. expected and actual output;
5. DAG graph screenshot or exported view;
6. task logs;
7. controlled failure evidence;
8. README with Airflow/Python versions and validation status.

## Completion criteria

The workflow is discoverable by Airflow, its tasks and dependencies are clear, the successful run is visible, the controlled failure blocks downstream work, and the student can explain the difference between syntax testing and Airflow runtime testing.
