# Day 44 Project Workbook — Sales Data Mart

## Goal

Translate a sales team's reporting requirements into a subject-oriented data mart.

## Step 1 — Identify users and decisions

List the users:
- sales manager;
- category manager;
- regional lead;
- finance analyst.

For each user, write one decision supported by the mart.

## Step 2 — Confirm metric definitions

Complete:
- paid sales;
- paid order count;
- units sold;
- refund amount;
- average order value.

For each metric, document grain, date role, inclusion/exclusion rules and owner approval.

## Step 3 — Identify model objects

Candidate fact models:
- sales order-line fact;
- refund event fact.

Candidate dimensions:
- date;
- product;
- customer;
- sales channel.

Explain why refund events should not be blindly joined to sales-line rows.

## Step 4 — Design the mart

Draw a diagram showing fact and dimension relationships. Mark the grain of each fact.

## Step 5 — Create the reporting model

Use the Day 44 SQL view from the class notes or adapt it to your Day 41 schema. Run the query and save its output.

## Step 6 — Validate metrics

Check:
- category sales totals;
- month totals;
- order count using distinct order IDs;
- sales line count;
- units sold;
- source-to-mart totals.

## Step 7 — Document ownership

Record:
- business owner;
- technical owner;
- refresh schedule;
- source models;
- data quality checks;
- security classification;
- known limitations.

## Step 8 — Failure drills

Explain what you would do if:
1. Finance and Sales report different revenue.
2. The mart refreshes but one source is missing.
3. Category sales double after adding a refund join.
4. The business changes the definition of paid sales.
5. A BI report calculates its own conflicting order count.

## Submission

- user and decision list;
- metric contract;
- mart diagram;
- SQL model;
- output evidence;
- validation evidence;
- ownership and documentation page.
