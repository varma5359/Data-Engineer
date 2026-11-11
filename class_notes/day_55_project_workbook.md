# Day 55 Project Workbook — Spark Data Exploration

## Goal

Create a local Spark session, read a small orders dataset and inspect its schema and contents. This is a local learning exercise, not a distributed cluster deployment.

## Step 1 — Check your environment

Record:
- operating system;
- Python version;
- Java version;
- PySpark version.

Install PySpark in the active Python environment. Check the selected Spark release's Java compatibility before troubleshooting.

## Step 2 — Create the dataset

Create the spark_day55 folder and orders.csv file using the exact five-row sample in the class notes.

## Step 3 — Create a SparkSession

Use local[*] and give the application a meaningful name. Record the Spark version.

Explain why local mode is not a multi-machine cluster.

## Step 4 — Read the CSV

Read with header=True and inferSchema=True.

Capture:
- successful load message;
- show() output;
- printSchema() output;
- columns;
- dtypes;
- count;
- describe(amount).

## Step 5 — Define an explicit schema

Create a schema with:
- order_id as integer;
- customer_id as string;
- city as string;
- amount as decimal(10,2).

Read the same file with the explicit schema and compare the schema output.

## Step 6 — Investigate a data-quality issue

Create a copy of the CSV with one invalid amount value such as the text NOT_AVAILABLE.

Read the file with the explicit schema. Inspect how your Spark version and read options handle the invalid value. Record the observed behavior and explain why production pipelines need parsing and validation rules.

Do not assume every Spark version or CSV mode handles malformed data identically.

## Step 7 — Explain architecture

Draw the flow:

Python code -> Spark driver -> cluster manager -> executors -> partitions

For local mode, clearly mark which components are local.

## Step 8 — Resource-safety check

Explain why show() with a small number of rows is safer than printing an entire dataset, and why collect() should not be used on a large dataset.

## Step 9 — Clean up

Stop the Spark session after the script or notebook work is complete. Keep the source CSV and evidence.

## Submission

- environment information;
- source CSV;
- code or notebook;
- show and schema outputs;
- count and describe output;
- explicit schema comparison;
- malformed-value observation;
- architecture diagram;
- short explanation of local versus cluster execution.

## Completion criteria

You can start Spark locally, read a CSV, explain the schema, inspect a sample and describe the role of the driver and executors without claiming that a cluster was deployed.
