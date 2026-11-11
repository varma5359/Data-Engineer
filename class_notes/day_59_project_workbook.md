# Day 59 Project Workbook — Cloud Pipeline Architecture

## Goal

Design an end-to-end batch pipeline and compare it with a streaming alternative. Run the local Spark simulation and document what additional work is required for a real cloud deployment.

## Scenario

TechStore needs a daily sales report by 7:00 AM. Order files arrive daily. Customer and product reference data are available from approved sources. The pipeline must preserve raw files, validate data, transform it with Spark and publish curated output only after successful checks.

## Step 1 — Write requirements

Document:
- source systems;
- file formats;
- arrival times;
- expected volume (mark assumptions);
- reporting deadline;
- business grain;
- unique keys;
- sensitive data;
- late-arrival behavior;
- recovery expectations.

## Step 2 — Draw the batch architecture

Include:
- source systems;
- ingestion;
- S3 raw zone;
- Spark processing;
- processed zone;
- data quality and reconciliation;
- curated zone;
- warehouse/BI;
- monitoring;
- IAM and network security.

Describe the responsibility of each component.

## Step 3 — Define the data contract

Document:
- required columns;
- data types;
- nullable rules;
- unique key;
- date semantics;
- amount rules;
- accepted/rejected record handling;
- schema-change process.

## Step 4 — Design idempotency

Describe how a rerun behaves if:
- Spark fails before writing;
- Spark writes some files then fails;
- output is written but run status is not updated;
- the same source file arrives twice.

Choose a run ID and a safe output publishing strategy.

## Step 5 — Run the local Spark simulation

Use the Day 59 notes:
1. create orders.csv;
2. read it with Spark;
3. validate required columns;
4. cast fields;
5. identify invalid rows;
6. aggregate daily sales;
7. validate totals;
8. write processed and curated Parquet;
9. read back the curated output.

Record actual output and environment details.

## Step 6 — Design monitoring

Define metrics:
- files received;
- input rows;
- accepted rows;
- rejected rows;
- output rows;
- processing duration;
- last successful run;
- failed run count;
- consumer lag if streaming is selected.

Define who receives alerts and what action follows.

## Step 7 — Design security

Document:
- ingestion role;
- Spark workload role;
- curated reporting role;
- raw-data access restrictions;
- encryption;
- network access;
- secret management;
- audit logging.

## Step 8 — Compare batch and streaming

Create a comparison for:
- latency requirement;
- event volume;
- state;
- late events;
- operational complexity;
- replay;
- cost.

Explain whether TechStore needs daily batch, micro-batch or streaming. Defend the decision using the 7:00 AM report requirement.

## Step 9 — Kafka fundamentals

Draw a Kafka flow with:
- producer;
- topic;
- partitions;
- brokers;
- consumer group;
- consumer;
- offsets.

Explain ordering within a partition and why consumer lag matters.

## Step 10 — Cloud integration checklist

For a future real deployment, list:
- S3 connector compatibility;
- IAM role;
- region and network;
- encryption/KMS permissions;
- Spark/Hadoop versions;
- output commit strategy;
- logging and monitoring;
- cleanup and cost controls.

Do not claim S3 integration if you only ran local paths.

## Step 11 — Submission

Submit:
1. requirements and data contract;
2. batch architecture;
3. local Spark code and outputs;
4. reconciliation evidence;
5. idempotency and publish design;
6. monitoring plan;
7. security matrix;
8. batch-versus-streaming comparison;
9. Kafka diagram;
10. cloud deployment checklist;
11. README with actual validation status.

## Completion criteria

A reviewer should understand the complete data flow, how records are validated, how Spark transforms them, how completed output is published, how reruns avoid duplicates, how failures are monitored and why batch or streaming was selected.
