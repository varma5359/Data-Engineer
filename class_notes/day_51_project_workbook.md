# Day 51 Project Workbook — Cloud Data Lake Layout

## Goal

Design and build a local example of a data lake layout, then document how the same design would map to object storage. The local exercise does not create cloud resources.

## Scenario

TechStore receives daily orders, payments and delivery files. It needs raw retention, cleaned datasets and curated reporting outputs.

## Step 1 — Create a dataset inventory

For each dataset record:
- dataset name;
- source system;
- owner;
- format;
- arrival frequency;
- approximate size (mark as estimate);
- sensitivity;
- expected consumers.

## Step 2 — Design the zones

Create a table describing what belongs in:
- raw;
- processed;
- curated;
- quarantine.

Explain which transformations are allowed in each zone.

## Step 3 — Design object keys

Create example keys for:
- order file received on 10 November 2026;
- payment file received on 10 November 2026;
- processed orders for business date 10 November 2026;
- curated daily sales for that date;
- rejected order records for that ingestion date.

Use a consistent convention and distinguish event date from ingestion date.

## Step 4 — Create the local folder structure

Create the structure from the class notes. Add the sample orders CSV.

## Step 5 — Run the Python exercise

Run the pandas steps:
1. read the CSV;
2. inspect columns, dtypes and shape;
3. convert order_date;
4. write Parquet;
5. read Parquet;
6. aggregate daily sales;
7. save the curated CSV.

Record actual outputs. If the Parquet engine is missing, document the error and install the required package in the active environment.

## Step 6 — Select file formats

For each dataset, choose CSV, JSON, Parquet or Avro and explain:
- source compatibility;
- nested data needs;
- analytical query pattern;
- schema needs;
- downstream tools.

There may be more than one reasonable answer.

## Step 7 — Partition strategy

Propose partition columns for orders and sales. Explain:
- common query filters;
- expected number of partitions;
- expected file sizes;
- late-arriving records;
- whether ingestion date and event date are both needed.

Avoid creating a partition for every customer without evidence.

## Step 8 — Governance and lifecycle

Document:
- who can read raw data;
- who can write processed data;
- who can read curated data;
- how rejected data is handled;
- how long each zone is retained;
- who approves deletion;
- how schema changes are communicated.

## Step 9 — Submission evidence

Submit:
- dataset inventory;
- zone definitions;
- object-key examples;
- local folder tree;
- screenshots or copied outputs from the Python exercise;
- file-format decisions;
- partition decision;
- access and retention rules;
- a short README.

## Completion criteria

A new engineer should be able to find a dataset, understand its source and date meaning, know which zone to read, identify its format and understand its access and retention rules.
