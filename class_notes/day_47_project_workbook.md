# Day 47 Project Workbook — Data Dictionary Generator

## Goal

Create a small catalog and dictionary for the TechStore sales warehouse.

## Step 1 — Select assets

Document:
- fact_sales;
- dim_customer;
- dim_product;
- dim_date.

## Step 2 — Create asset records

For each asset, record:
- name;
- business purpose;
- grain;
- owner;
- refresh frequency;
- classification;
- upstream sources.

## Step 3 — Create column definitions

For at least ten columns, record:
- column name;
- type;
- required/optional;
- business definition;
- example value;
- unit/currency;
- sensitivity;
- key role.

## Step 4 — Create a glossary

Define:
- paid sales;
- order line;
- paid order;
- refund;
- customer;
- product category.

Ask the relevant business owner to approve definitions.

## Step 5 — Record lineage

Trace three important columns from source to presentation model.

## Step 6 — Inspect PostgreSQL metadata

Use information_schema to retrieve the structure of a permanent training table. Explain which metadata it provides and which business metadata it cannot provide.

## Step 7 — Submission

- catalog register;
- data dictionary;
- business glossary;
- ownership register;
- classification register;
- lineage entries;
- SQL used to inspect structural metadata.

Use synthetic examples only.
