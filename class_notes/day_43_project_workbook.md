# Day 43 Project Workbook — Warehouse Key Management

## Goal

Design and validate key relationships for a multi-source warehouse.

## Business scenario

TechStore receives customers from CRM and Loyalty. Both systems may use the same numeric customer ID. Orders arrive with an order ID and line number.

## Step 1 — Identify source identity

Document the source system and source key together.

| Source | Source customer ID | Warehouse identity |
|---|---|---|
| CRM | 501 | CRM + 501 |
| Loyalty | 501 | Loyalty + 501 |

Do not assume the two records are the same person.

## Step 2 — Choose warehouse keys

For each entity, decide:
- natural key;
- surrogate key;
- candidate key;
- alternate key;
- composite key if applicable.

Explain the reason for each choice.

## Step 3 — Build the SQL examples

Use the class-note SQL to create:
- generated surrogate dimension key;
- primary key;
- foreign key relationship;
- composite order-line key.

Capture successful insert outputs and one rejected duplicate/invalid-key attempt.

## Step 4 — Validate

Run:
- duplicate source identity check;
- duplicate composite key check;
- orphan relationship check.

Expected exception-query results: zero rows.

## Step 5 — Unknown member policy

Write a short rule for what happens when a fact arrives before its dimension.

Include:
- reserved key;
- reason code;
- monitoring count;
- resolution process;
- fact correction process if needed.

## Step 6 — Submission

- key strategy table;
- SQL script;
- successful and rejected outputs;
- relationship diagram;
- unknown-member policy;
- README explaining source identity and assumptions.

Never include real customer personal data.
