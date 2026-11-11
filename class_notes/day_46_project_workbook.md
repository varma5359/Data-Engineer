# Day 46 Project Workbook — Source-to-Target Reconciliation

## Goal

Compare a source dataset with its warehouse target and produce evidence of correctness.

## Step 1 — Define the load scope

Document:
- source table;
- target table;
- business key;
- eligible status values;
- date range;
- time zone;
- expected transformation.

## Step 2 — Create the training tables

Use the Day 46 class-note SQL. Load the source rows and eligible target rows.

## Step 3 — Run reconciliation

Capture:
- source eligible count;
- target count;
- distinct key counts;
- missing source keys;
- extra target keys;
- source amount total;
- target amount total;
- grouped totals by date.

## Step 4 — Inject a mismatch

Change one target amount using the class-note update. Run the field-level comparison and capture the mismatched order.

Restore the training value after the test.

## Step 5 — Build a mapping register

Document source field, target field, transformation, type, null rule, key role and business meaning.

## Step 6 — Build a lineage register

Trace at least three output columns from source to target and report.

## Step 7 — Write an incident note

Use:
- symptom;
- impact;
- scope;
- evidence;
- root cause;
- correction;
- retest result;
- prevention.

## Submission

- SQL script;
- reconciliation outputs;
- mismatch evidence;
- source-to-target mapping;
- lineage register;
- incident note.
