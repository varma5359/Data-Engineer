# Day 58 Interview Guide — Spark Performance Optimization

## 1. How do you investigate a slow Spark job?

I capture a baseline, inspect the execution plan, examine stage/task duration, shuffle read/write, spill, partition sizes and data skew. I identify the bottleneck before changing code or cluster size. I then change one thing, measure again and validate correctness.

## 2. What does explain() do?

explain() displays the DataFrame execution plan. Depending on the mode, it can show logical and physical plan information. It helps identify scans, filters, joins, aggregations, exchanges and other operations.

## 3. What is a narrow transformation?

A narrow transformation can usually process an output partition using data from one input partition without redistributing records across the cluster. Filters and simple projections are common examples.

## 4. What is a wide transformation?

A wide transformation needs data from multiple input partitions to be redistributed or combined. Grouping, distinct, repartition, some joins and global sorting can require shuffle.

## 5. What is shuffle and why is it expensive?

Shuffle redistributes data between workers. It can require serialization, network transfer, memory buffers, disk writes and reads. Large shuffle volumes can increase runtime and resource cost.

## 6. What is caching?

Caching stores computed data so subsequent operations can reuse it instead of recomputing the lineage. It is useful when an expensive DataFrame is reused, but it consumes memory or other storage resources.

## 7. When should you unpersist?

When the cached or persisted DataFrame is no longer needed. Releasing it helps return resources to Spark.

## 8. What is partition pruning?

Partition pruning skips storage partitions that cannot match a query filter. It reduces data scanned when the query predicate aligns with the partition layout and the source/engine supports pruning.

## 9. What is data skew?

Data skew is uneven distribution of records across keys or partitions. Some tasks process much more data and become stragglers. I inspect partition sizes and task duration and consider better keys, pre-aggregation or skew-aware engine features.

## 10. What is a broadcast join?

A broadcast join sends a small dataset to executors so each can join it locally with a partition of a larger dataset. It can avoid shuffling both sides, but broadcasting a large dataset can exhaust executor memory.

## 11. Why are built-in Spark functions often preferred to Python UDFs?

Built-in expressions are understood by Spark's optimizer and can execute efficiently in the engine. Python UDFs may add serialization and Python execution overhead and can limit some optimizations. I use UDFs only when built-in expressions cannot express the required logic.

## 12. What is the small-file problem?

Too many tiny files increase metadata listing, file-open and task-scheduling overhead. Compaction and appropriate output partitioning can help, but file sizing must preserve parallelism and recovery needs.

## 13. What is spill?

Spill occurs when intermediate data cannot remain in memory and is written to disk. Disk spill can slow a job. I inspect memory pressure, partition sizes, shuffle and aggregation behavior.

## 14. Does selecting fewer columns always improve performance?

It can reduce data read and moved, but Spark may already perform column pruning. I inspect the plan and measure rather than assuming the change improved runtime.

## 15. Why is one timing not enough?

Timing varies with cache state, concurrency, cluster startup, CPU, network and other conditions. I use comparable input and configuration, repeat tests where appropriate and record the environment.

## 16. How do you validate an optimization?

I compare row counts, business keys, important totals, null behavior, output schema and filter semantics. A faster query is not acceptable if it changes the intended business result.

## 17. Scenario: a groupBy job has one task much slower than the others.

I would inspect task input sizes, key distribution, shuffle metrics and partition sizes. A heavily skewed key may be responsible. I would consider skew-aware processing, a better partition strategy or pre-aggregation, then validate the result.

## 18. Scenario: a DataFrame is used by four downstream actions.

If it is expensive to compute and reused, caching or persistence may avoid recomputation. I would measure the cost and memory impact, materialize it when needed and unpersist after use.

## 19. Scenario: a join has a small lookup table and a huge fact table.

A broadcast join may be appropriate if the lookup table is small enough for executor memory. I would inspect the plan and size rather than forcing broadcast blindly.

## 20. Explain your Day 58 performance experiment.

I created a synthetic dataset, inspected a groupBy execution plan, measured a baseline, selected only required columns as a candidate, compared plans and timings, and validated group counts and totals. I documented that Spark may already prune columns, so no improvement should be claimed unless measured.

## Common interview traps

- Increasing cluster size without inspecting the plan.
- Treating every shuffle as avoidable.
- Caching every DataFrame.
- Forgetting to unpersist.
- Broadcasting a large dataset.
- Ignoring skew and spill.
- Using UDFs for simple built-in logic.
- Claiming speedups without measurements.
- Optimizing without validating correctness.

## Readiness checklist

- [ ] Explain execution plans.
- [ ] Explain narrow/wide transformations and shuffle.
- [ ] Explain cache, persist and unpersist.
- [ ] Explain partition pruning.
- [ ] Identify skew and stragglers.
- [ ] Explain broadcast joins.
- [ ] Explain UDF trade-offs and small files.
- [ ] Describe a measured optimization workflow.
