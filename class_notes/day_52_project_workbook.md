# Day 52 Project Workbook — Object Storage Organizer

## Goal

Design an S3 storage layout and document safe object operations. Do not create paid resources unless you have an approved training account and have checked current pricing.

## Scenario

TechStore receives daily order exports. The platform must retain each delivery, keep raw data private, move approved files through processing and retain curated reports.

## Step 1 — Create the bucket plan

Document:
- bucket name (example only until availability is checked);
- region;
- environment;
- owner;
- sensitivity classification;
- encryption requirement;
- public access posture;
- versioning decision;
- retention requirements.

## Step 2 — Design prefixes

Create a prefix layout for:
- raw orders;
- processed orders;
- curated daily sales;
- quarantine;
- archive.

Include ingestion date in raw object keys. Explain why event date may be a separate field.

## Step 3 — Plan object naming

Create object key examples for three consecutive daily deliveries. Explain how your design prevents accidental overwrite.

## Step 4 — CLI command review

In the class notes, inspect:
- aws sts get-caller-identity;
- aws s3 ls;
- aws s3 cp;
- aws s3 sync.

For each command, identify source, destination, identity requirements and any destructive behavior.

Do not execute commands using a bucket you do not own or have permission to access.

## Step 5 — Storage class decision

Choose a storage class family for:
- frequently queried curated reports;
- old audit files accessed rarely;
- temporary export files;
- irreplaceable source data.

Explain retrieval delay, retrieval charges, minimum storage duration and recovery needs. Check current AWS pricing and service documentation before making a real decision.

## Step 6 — Lifecycle proposal

Write a lifecycle proposal. Specify which prefix or tags are in scope, what transitions or expiration actions occur, and who approved the retention period.

Do not invent retention periods as legal requirements.

## Step 7 — Security design

Document:
- Block Public Access posture;
- encryption at rest;
- secure transport;
- IAM role responsibilities;
- bucket policy boundaries;
- KMS permissions if customer-managed keys are used.

## Step 8 — Versioning and recovery

Decide whether versioning is needed. Explain how it helps with accidental overwrite and how noncurrent versions affect storage cost.

## Step 9 — Operational checklist

Create pre-upload, post-upload and pre-delete checklists. Include identity verification, object size, checksum where appropriate, approval and logging.

## Step 10 — Submission

Submit:
1. bucket plan;
2. prefix layout;
3. object naming examples;
4. storage class decision;
5. lifecycle proposal;
6. security matrix;
7. versioning/recovery decision;
8. CLI command explanations;
9. cleanup and cost checklist.

## Completion criteria

A reviewer should be able to understand how objects are named, who can access them, how they are protected, when they move or expire and how accidental changes are recovered.
