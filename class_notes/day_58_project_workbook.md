# Day 58 Project Workbook — Spark Performance Experiment

## Goal

Inspect a Spark execution plan, establish a baseline, test a candidate change and validate that the output remains correct.

## Step 1 — Record the environment

Document:
- Spark version;
- Java version;
- local or cluster mode;
- CPU/memory configuration where known;
- input row count and size;
- cache state.

## Step 2 — Create the synthetic dataset

Run the Day 58 range-based example to create 500,000 rows with customer_id and amount.

## Step 3 — Capture a baseline plan

Group sales by customer_id. Run explain("formatted") and save the plan output.

Identify:
- source;
- aggregation;
- exchange/shuffle;
- output.

## Step 4 — Measure baseline

Run the aggregation action and record actual time. Do not copy another environment's timing.

## Step 5 — Build a candidate

Select only the required columns before grouping. Inspect the candidate plan.

Note that Spark may already perform column pruning, so the candidate may have the same physical plan and similar time. That is a valid finding.

## Step 6 — Validate correctness

Compare:
- group count;
- source amount total;
- grouped amount total;
- schema;
- any important business keys.

The results must remain equivalent.

## Step 7 — Experiment with cache

Cache a DataFrame only if it is reused. Measure first materialization and subsequent reuse separately. Call unpersist after the experiment.

Explain why a first cache action and a later cache action are not directly comparable.

## Step 8 — Review optimization concepts

Write a short explanation of:
- shuffle;
- partition pruning;
- data skew;
- broadcast join;
- small-file problem;
- built-in expressions versus Python UDFs.

## Step 9 — Performance decision record

For each proposed optimization, document:
- problem evidence;
- proposed change;
- expected benefit;
- risk;
- actual result;
- correctness validation;
- decision.

## Step 10 — Submission

Submit:
1. environment details;
2. baseline plan;
3. candidate plan;
4. measured timings;
5. correctness comparison;
6. cache experiment;
7. optimization decision record;
8. README with limitations.

## Completion criteria

You can explain why a query is expensive using a plan, make one controlled change, measure it fairly and demonstrate that the business result did not change.
