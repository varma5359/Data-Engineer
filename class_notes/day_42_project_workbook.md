# Day 42 Project Workbook — Customer History Model

## Goal

Implement and validate a customer dimension that preserves city history using SCD Type 2. Compare it with Type 1 and Type 3.

## Business requirement

TechStore needs to answer:
- What is the customer's current city?
- Which city was valid when an order occurred?
- How many customer city changes have been recorded?

## Step 1 — Confirm the history rule

Document:
- business key: customer_id;
- tracked attribute: city;
- history strategy: Type 2;
- effective-date convention: inclusive dates for this exercise;
- current row indicator: is_current.

## Step 2 — Create the dimension

Use the PostgreSQL Type 2 table definition from the class notes.

## Step 3 — Load the first version

Insert Asha in Hyderabad from 2026-01-10.

Capture the output and verify one current row.

## Step 4 — Apply a change

Close the Hyderabad version on 2026-04-14 and insert Bengaluru from 2026-04-15.

Capture the SQL outputs.

## Step 5 — Query historical values

Verify:
- 2026-03-01 returns Hyderabad;
- 2026-05-01 returns Bengaluru.

Each lookup must return exactly one row.

## Step 6 — Validate the dimension

Run:
- duplicate current-row check;
- overlap check;
- historical date lookup;
- current-row lookup.

Expected exception-query result: zero rows.

## Step 7 — Compare SCD types

Create a short table explaining:
- what Type 1 stores;
- what Type 2 stores;
- what Type 3 stores;
- which business requirement each satisfies.

## Step 8 — Failure drills

Explain how you would handle:
1. Two current rows for customer 501.
2. No current row after a failed load.
3. A city change received ten days late.
4. A daily load with no actual city change.
5. A fact that cannot find a valid dimension version.

## Submission

- SQL script;
- before/after query outputs;
- historical lookup outputs;
- validation outputs;
- SCD comparison;
- short README describing assumptions.

Do not use real customer personal information.
