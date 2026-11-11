# Day 50 Project Workbook — Cloud Architecture Design

## Project goal

Design a secure, understandable cloud data architecture for a small online retail company. This is a design exercise. You do not need to create paid AWS resources to complete it.

## Business scenario

TechStore receives:
- daily order exports from its order application;
- customer updates from a customer application;
- payment status files from a payment provider;
- delivery status files from a logistics partner.

The business needs a daily sales report by 8:00 AM. Analysts should read approved reporting data. Raw customer information must not be accessible to every user.

## Step 1 — Write the requirements

Create a requirements document with:
- source systems;
- data formats;
- approximate daily volume (mark estimates clearly);
- arrival frequency;
- reporting deadline;
- sensitive fields;
- recovery expectations;
- business owner and technical owner.

Do not invent a precise production volume. If you need a practice number, label it as an assumption.

## Step 2 — Draw the architecture

Use this flow as a starting point:

Source systems -> Ingestion -> Raw storage -> Processing -> Curated storage -> Warehouse/BI

Add monitoring and identity/access controls across the workflow.

For each box, write what it receives, what it produces, who owns it and how failure is detected.

## Step 3 — Classify service models

For each proposed capability, explain whether it is closest to IaaS, PaaS or SaaS.

Include at least:
- a virtual machine running a Python extraction script;
- a managed application runtime;
- a hosted reporting application.

Explain why the classification is appropriate.

## Step 4 — Choose a deployment approach

Describe whether the scenario uses public cloud, private cloud or a hybrid design. State your assumptions. There is no single correct answer without requirements.

## Step 5 — Select a region and availability approach

Write down the factors that would influence region selection:
- customer location and latency;
- data residency;
- service availability;
- recovery requirements;
- cost.

Explain whether a multi-AZ design is needed for each critical component. Do not claim that multi-AZ protects against every type of failure.

## Step 6 — Create an IAM permission matrix

Complete this matrix:

| Identity | Required actions | Resources | Actions not required |
|---|---|---|---|
| Data engineer |  |  |  |
| Pipeline role |  |  |  |
| Reporting analyst |  |  |  |

Use least privilege. Separate the permissions required to read raw data from those required to read curated reports.

## Step 7 — Review the sample policy

In the class notes, inspect the sample policy that allows s3:GetObject on an example object prefix.

Answer:
1. What action is allowed?
2. What resource path is in scope?
3. Does it grant bucket listing?
4. Does it grant object deletion?
5. Why must a real policy be checked against the actual task and service authorization rules?

## Step 8 — Create a threat and risk register

Include at least five risks:
- leaked access key;
- public data exposure;
- overly broad IAM policy;
- database network open to everyone;
- forgotten paid cloud resource;
- insufficient backup or recovery plan.

For each, document likelihood, impact, prevention and response.

## Step 9 — Create a cost-safety checklist

Before any optional cloud deployment:
- review current pricing;
- understand free-tier eligibility and limits;
- choose a region;
- set a budget alert where available;
- record resource names;
- avoid sensitive real data;
- define cleanup steps;
- verify deletion and review billing.

Remember: an alert is not necessarily a hard spending limit.

## Step 10 — Submit your evidence

Your submission should contain:
1. business requirements;
2. architecture diagram;
3. service-model comparison;
4. region and availability decision;
5. IAM permission matrix;
6. risk register;
7. cost-safety checklist;
8. short explanation of design trade-offs.

## Completion criteria

You are finished when another beginner can read your design and understand where source data enters, where it is stored, how it is processed, who can access it, how failures are noticed and how cost is controlled.
