# Day 48 Project Workbook — Warehouse Load Simulator

## Goal

Compare full, append and upsert loading and document a safe incremental strategy.

## Step 1 — Define source behavior

Choose a training dataset and document:
- business key;
- whether records are immutable;
- whether updates occur;
- whether deletes occur;
- update timestamp or sequence;
- late-arrival behavior.

## Step 2 — Full load

Use the class-note product example. Load all source rows and capture counts.

## Step 3 — Append

Insert a new event. Attempt a duplicate event key and capture the database response.

Explain why append needs replay protection.

## Step 4 — Upsert

Update an existing product and insert a new product using PostgreSQL ON CONFLICT.

Capture before/after rows.

## Step 5 — Incremental design

Write pseudocode for:
- reading from the last safe watermark;
- handling timestamp ties;
- validating staging;
- upserting target;
- reconciling;
- advancing checkpoint only after success.

## Step 6 — Choose strategy

Complete a table for:
- country reference;
- immutable click events;
- mutable product current state;
- customer history;
- corrected monthly aggregate.

## Step 7 — Failure drills

Explain:
1. Target write succeeds but checkpoint save fails.
2. The same source batch is replayed.
3. Two source rows have the same updated_at timestamp.
4. A source row is deleted.
5. A partition replacement contains only half the expected data.

## Submission

- SQL scripts;
- output evidence;
- load strategy decision table;
- watermark and replay design;
- validation checklist;
- README with assumptions.
