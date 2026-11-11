# Day 54 Project Workbook — Parallel Processing Experiment

## Goal

Demonstrate how a dataset can be divided into partitions, calculate partial results and combine them. Then document what changes when the work is distributed across machines.

## Scenario

TechStore wants to calculate total sales from a large set of transactions. The training dataset is small, but the design should explain how the same logical work could be divided across workers.

## Step 1 — Define the workload

Document:
- input dataset;
- expected output;
- operation (sum, count or group total);
- correctness condition;
- approximate input size;
- whether a local process is sufficient.

## Step 2 — Run the local simulation

Use the Day 54 Python list example:
1. create the sample sales list;
2. calculate the original total;
3. split the list into three partitions;
4. calculate partial totals;
5. combine partial totals;
6. compare the combined result with the original total.

Record actual output.

## Step 3 — Introduce a partition error

Run the example that skips one value. Explain why the result differs and how row-count or key-level validation could catch the problem.

## Step 4 — Design a worker assignment

Create a table:

| Partition | Input records | Assigned worker | Expected output |
|---|---|---|---|
| P1 |  | Worker A |  |
| P2 |  | Worker B |  |
| P3 |  | Worker C |  |

The worker names are logical placeholders. You are designing a distributed execution, not deploying three servers.

## Step 5 — Identify bottlenecks

Describe how each issue could affect the job:
- too few partitions;
- too many tiny partitions;
- slow network;
- shuffle;
- data skew;
- one worker failure;
- non-idempotent output write.

## Step 6 — Design retry safety

Describe how a task can be retried without:
- double-counting;
- duplicating output;
- exposing partial files;
- incrementing external state twice.

## Step 7 — Compare scaling options

Explain when you would:
- increase one machine's CPU or memory;
- add more workers;
- keep the workload local.

Include operational complexity and cost in the decision.

## Step 8 — Submission

Submit:
1. workload definition;
2. local code and actual output;
3. partition table;
4. correctness comparison;
5. bottleneck analysis;
6. retry/idempotency design;
7. scaling decision;
8. short architecture diagram.

## Completion criteria

You can explain how input is split, how workers process independent tasks, where data movement happens, how partial results are combined and how the final result is validated.
