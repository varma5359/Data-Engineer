# Day 54: Distributed Computing

**Phase:** 6 — Cloud and Distributed Processing  
**Learning time:** 2–3 hours  
**Audience:** Complete beginners and junior Data Engineer candidates  
**Topic boundary:** Class notes only. Project workbook and interview guide are separate files.

> A laptop can process small datasets, but a company may need to process billions of events. One machine has limits in CPU, memory, storage and time. Distributed computing divides work across multiple machines or workers. Today you will learn how work is split, how results are combined, what partitions and shuffles mean, and why distributed systems introduce new failure and data-correctness challenges.

## Learning outcomes

- Explain distributed computing.
- Compare sequential, parallel and distributed processing.
- Explain nodes, workers, clusters and tasks.
- Understand data partitioning.
- Explain horizontal scaling and vertical scaling.
- Understand shuffle and data movement.
- Explain fault tolerance and retries.
- Recognize data skew and stragglers.
- Understand data locality and network overhead.
- Identify when distributed processing is unnecessary.
- Perform a simple local simulation of partitioned work.

---

# Part 1 — Why do we need distributed computing?

## 1. Start with a business problem

TechStore receives customer activity events from its website and mobile application.

One day of data contains:
- page views;
- product searches;
- cart additions;
- orders;
- payment events;
- delivery events.

Suppose the data grows from thousands of rows to billions of events.

A small laptop may not have enough memory or CPU to process the full dataset efficiently.

The team needs a way to divide the work.

## 2. What is distributed computing?

**Beginner definition:** Distributed computing means using multiple connected computers or workers to complete parts of a workload and coordinate the results.

Instead of one computer doing every task, a distributed system divides work across multiple machines.

The machines communicate through a network.

A distributed data processing job may:
1. read input data;
2. divide data into partitions;
3. send partitions to workers;
4. process partitions in parallel;
5. exchange data when required;
6. combine partial results;
7. write final output.

## 3. Sequential processing

Sequential processing performs operations one after another.

Example:

~~~text
Read file
   |
Process all rows
   |
Calculate total
   |
Write result
~~~

For small data, sequential processing is often simple and efficient.

## 4. Parallel processing

Parallel processing performs multiple independent tasks at the same time.

Example:
- Task A processes file A;
- Task B processes file B;
- Task C processes file C.

The tasks may run on different CPU cores of one computer or on different computers.

Parallelism does not automatically mean distributed computing. Multiple CPU cores inside one machine can run parallel tasks.

## 5. Distributed processing

Distributed processing uses multiple networked machines or workers.

Example:

~~~text
                 Coordinator
                /     |      \
             Worker  Worker  Worker
                \     |      /
                 Combined result
~~~

The workers may have separate CPU, memory and local storage.

The system must coordinate task execution, data movement, failures and final results.

## 6. Compare sequential, parallel and distributed

| Approach | Where work runs | Main idea |
|---|---|---|
| Sequential | One execution path | One step after another |
| Parallel | Multiple cores or workers | Several tasks at once |
| Distributed | Multiple networked machines | Coordinate work across machines |
| Hybrid | Multiple machines and multiple cores | Distributed and parallel execution together |

---

# Part 2 — Understand the building blocks

## 7. What is a node?

A node is a computer or machine participating in a distributed system.

A node may provide:
- CPU;
- memory;
- local disk;
- network connectivity.

A cluster may contain multiple nodes.

## 8. What is a worker?

A worker is a process or compute component that performs assigned work.

Depending on the platform, a worker may run on a node or inside a container.

A worker can process one or more tasks.

## 9. What is a cluster?

A cluster is a group of connected compute resources managed to work together.

A distributed data processing cluster may contain:
- a coordinator or driver;
- worker nodes;
- a cluster manager;
- storage access;
- network communication.

The exact architecture differs by platform. Apache Spark's architecture is introduced in Day 55.

## 10. What is a task?

A task is a unit of work assigned to a worker.

Examples:
- read one partition;
- filter a subset of records;
- calculate a partial sum;
- write one output partition.

A task should have a clear input and output.

## 11. What is a job?

A job is a larger unit of work submitted to a processing system.

A job may contain multiple stages and tasks.

For example, a daily sales aggregation job may:
1. read order files;
2. filter valid orders;
3. group by product;
4. calculate total sales;
5. write output.

---

# Part 3 — Understand scaling

## 12. What is vertical scaling?

Vertical scaling means increasing the capacity of one machine.

Examples:
- more CPU;
- more memory;
- faster storage.

Advantages:
- simpler architecture in many cases;
- fewer distributed coordination concerns;
- may be sufficient for moderate workloads.

Limitations:
- machine size has limits;
- larger machines can be expensive;
- one machine may remain a single failure point.

## 13. What is horizontal scaling?

Horizontal scaling means adding more machines or workers.

Example:
- start with two workers;
- add two more workers;
- distribute eligible tasks across the larger worker pool.

Advantages:
- can increase processing capacity;
- can support large workloads;
- can improve resilience when designed correctly.

Limitations:
- network communication;
- coordination overhead;
- data movement;
- skew;
- operational complexity;
- not every task can be divided efficiently.

## 14. Vertical versus horizontal scaling

| Feature | Vertical scaling | Horizontal scaling |
|---|---|---|
| Change | Increase one machine's resources | Add more machines/workers |
| Complexity | Often simpler | Requires coordination |
| Limit | Machine capacity | Cluster/service limits |
| Main concern | Large machine cost | Network and distributed overhead |
| Example | More memory in one server | More Spark executors |

## 15. Does adding workers always make a job faster?

No.

Suppose a job spends most of its time waiting for one external API. Adding more workers may not help and may overload the API.

Other bottlenecks include:
- reading storage;
- network bandwidth;
- data skew;
- serial code;
- shuffle;
- disk spill;
- database limits;
- task startup overhead.

Measure before scaling.

---

# Part 4 — Divide data into partitions

## 16. What is a partition?

A partition is a subset of a dataset that can be processed as a unit.

Suppose there are 12 records.

They can be divided into three partitions:

| Partition | Record IDs |
|---|---|
| P1 | 1, 2, 3, 4 |
| P2 | 5, 6, 7, 8 |
| P3 | 9, 10, 11, 12 |

Workers can process these partitions independently if the operation allows it.

## 17. Why partition data?

Partitioning can:
- divide large work into smaller tasks;
- allow parallel processing;
- reduce memory requirements per task;
- support recovery and retries;
- improve data organization.

The partition strategy affects performance.

## 18. How many partitions should we create?

There is no universal number.

Consider:
- input size;
- worker count;
- CPU cores;
- task duration;
- memory;
- file sizes;
- scheduling overhead;
- downstream operations.

Too few partitions:
- workers may sit idle;
- tasks may be too large;
- parallelism may be limited.

Too many tiny partitions:
- scheduling overhead increases;
- task setup can dominate useful work;
- too many small output files may be created.

## 19. Partitioning is not the same as splitting randomly

A partition strategy can be based on:
- file blocks;
- row ranges;
- date values;
- hash of a key;
- storage partitions;
- processing engine rules.

The strategy should match the workload.

---

# Part 5 — Understand shuffle and data movement

## 20. What is shuffle?

Shuffle is the movement of data across workers so records that need to be processed together can be brought together.

Example: We have sales records distributed across workers. We need total sales per customer.

Customer C001 may appear on Worker A, Worker B and Worker C.

To calculate the final total for C001, the partial records or aggregates must be combined.

This often requires data movement.

## 21. Simple shuffle example

Input partitions:

| Worker A | Worker B |
|---|---|
| C001, 100 | C002, 200 |
| C002, 50 | C001, 300 |
| C001, 25 | C003, 75 |

Goal: group by customer_id.

After shuffle, records for the same customer are routed to the same destination partition.

| Destination group | Records |
|---|---|
| C001 | 100, 300, 25 |
| C002 | 200, 50 |
| C003 | 75 |

Then the system can calculate grouped totals.

## 22. Why can shuffle be expensive?

Shuffle may involve:
- serialization;
- network transfer;
- disk writes;
- disk reads;
- sorting or hashing;
- memory pressure.

Large shuffles can slow jobs.

Spark-specific shuffle behavior is studied later.

## 23. Operations that may cause shuffle

Depending on engine and execution plan, operations such as these may require data redistribution:
- groupBy;
- distinct;
- repartition;
- joins on keys;
- global sorting.

Not every operation creates a shuffle. A simple filter on an existing partition can often run within each partition.

---

# Part 6 — Data locality and network

## 24. What is data locality?

Data locality means processing data close to where it is stored, when the platform supports that design.

Moving large datasets across a network can be expensive and slow.

A distributed platform may try to schedule work near the data or use storage systems designed for parallel reads.

Cloud object storage is accessed over service APIs and networks. The compute and storage placement should be considered together.

## 25. Network overhead

Distributed workers communicate through a network.

Network overhead can arise from:
- reading remote data;
- shuffle;
- joining data from different sources;
- sending results to a coordinator;
- writing outputs.

A job can have enough CPU but still be slow because it moves too much data.

---

# Part 7 — Fault tolerance and reliability

## 26. What is fault tolerance?

Fault tolerance is the ability of a system to continue operating or recover when some component fails.

In distributed data processing, failures may include:
- worker process stops;
- node becomes unavailable;
- network interruption;
- storage request fails;
- task runs out of memory;
- input file is corrupt.

A distributed engine may retry failed tasks or reschedule work.

## 27. Why can tasks be retried?

If one worker fails while processing a partition, the system may run that partition's task again on another worker.

This is useful only when the operation is safe to retry.

## 28. Idempotency matters

An operation is idempotent when repeating it produces the same final effect as running it once, under the defined conditions.

Example:
Writing a complete output partition to a deterministic path using a safe overwrite strategy can be designed to be repeatable.

Dangerous example:
A task increments an external counter every time it runs. If the task is retried, the counter may be incremented twice.

For data pipelines, design writes and side effects so retries do not create duplicates or incorrect totals.

## 29. Partial output problem

Imagine a job writes 10 output files and fails after writing 6.

If downstream systems read the output folder immediately, they may consume incomplete data.

Safer patterns can include:
- write to a temporary/run-specific location;
- validate output;
- publish a completion marker or table commit;
- expose only completed output;
- clean up abandoned temporary data.

The exact method depends on storage format and processing platform.

---

# Part 8 — Data skew and stragglers

## 30. What is data skew?

Data skew occurs when data is distributed unevenly across partitions or workers.

Example:
- Worker A gets 1 million records;
- Worker B gets 1 million records;
- Worker C gets 90 million records.

Worker C takes much longer.

The job waits for the slowest task or stage.

## 31. What is a straggler?

A straggler is a task that takes much longer than other tasks in the same stage.

Possible causes:
- data skew;
- slow disk;
- slow network;
- worker resource contention;
- garbage collection or memory pressure;
- unusually complex records.

## 32. Why average task time can hide a problem

Suppose four tasks take:
- 10 seconds;
- 11 seconds;
- 12 seconds;
- 120 seconds.

The average is 38.25 seconds, but the real completion time is dominated by the task that takes 120 seconds.

Look at task distribution and the slowest tasks, not only averages.

## 33. How can skew be addressed?

Possible approaches:
- choose a better partition key;
- pre-aggregate before a large join where logically valid;
- use skew-aware engine features;
- separate exceptionally large keys;
- repartition carefully;
- salt keys in suitable cases.

Do not apply salting automatically. It changes key distribution and may require a second aggregation to preserve correctness.

---

# Part 9 — Consistency and distributed-system trade-offs

## 34. Why is distributed correctness harder?

A distributed job has multiple components working at the same time.

Challenges include:
- network delay;
- worker failure;
- retry;
- duplicate execution;
- partial writes;
- inconsistent source snapshots;
- late-arriving data;
- coordination overhead.

The pipeline design must define what correctness means.

## 35. Exactly-once is not automatic

A platform may provide delivery or processing guarantees under particular conditions, but a pipeline can still create duplicate effects when writing to external systems or retrying non-idempotent operations.

Understand:
- source delivery guarantees;
- checkpoint behavior;
- sink transaction support;
- idempotency keys;
- deduplication;
- commit protocol.

Do not claim "exactly once" without defining the end-to-end system boundary.

## 36. Distributed processing trade-offs

| Benefit | Trade-off |
|---|---|
| More compute capacity | More coordination |
| Parallel execution | Shuffle and network cost |
| Fault recovery | Retry and duplicate-effect concerns |
| Scale out | Cluster management |
| Large dataset support | Partition and skew tuning |
| Shared workload | Resource contention |

---

# Part 10 — Local simulation: split and combine work

This exercise shows partitioned work using a small Python list. It does not create a distributed cluster. It simulates the idea of dividing work into groups.

## 37. Create sample sales values

~~~python
sales = [100, 200, 150, 300, 250, 400, 50, 350, 500, 100, 200, 150]

print("Total records:", len(sales))
print("Total sales:", sum(sales))
~~~

Expected output:

~~~text
Total records: 12
Total sales: 2750
~~~

## 38. Divide into three partitions

~~~python
partition_1 = sales[0:4]
partition_2 = sales[4:8]
partition_3 = sales[8:12]

print(partition_1)
print(partition_2)
print(partition_3)
~~~

Expected output:

~~~text
[100, 200, 150, 300]
[250, 400, 50, 350]
[500, 100, 200, 150]
~~~

Each partition contains four values.

## 39. Calculate partial totals

~~~python
total_1 = sum(partition_1)
total_2 = sum(partition_2)
total_3 = sum(partition_3)

print(total_1)
print(total_2)
print(total_3)
~~~

Expected output:

~~~text
750
1050
950
~~~

These are partial results.

## 40. Combine partial totals

~~~python
distributed_style_total = total_1 + total_2 + total_3

print("Combined total:", distributed_style_total)
~~~

Expected output:

~~~text
Combined total: 2750
~~~

The result matches the original total.

This is a map/reduce-style idea:
- map-like step: calculate a partial result for each partition;
- reduce-like step: combine the partial results.

The code ran on one Python process. A real distributed engine schedules partitions across workers and handles network, failures and execution planning.

## 41. Verify correctness

~~~python
print("Original total:", sum(sales))
print("Partition total:", distributed_style_total)
print("Same result:", sum(sales) == distributed_style_total)
~~~

Expected output:

~~~text
Original total: 2750
Partition total: 2750
Same result: True
~~~

If the totals do not match, the partition ranges may have missed or repeated values.

## 42. Simulate a partitioning mistake

~~~python
wrong_partition_1 = sales[0:4]
wrong_partition_2 = sales[4:8]
wrong_partition_3 = sales[9:12]

wrong_total = (
    sum(wrong_partition_1)
    + sum(wrong_partition_2)
    + sum(wrong_partition_3)
)

print("Wrong total:", wrong_total)
print("Correct total:", sum(sales))
~~~

Expected output:

~~~text
Wrong total: 2650
Correct total: 2750
~~~

One record was skipped. This shows why partition boundaries and data completeness must be validated.

---

# Part 11 — When should you use distributed computing?

## 43. Use it when the workload justifies it

Distributed processing may be useful when:
- data does not fit comfortably on one machine;
- processing time exceeds the business window;
- workload can be divided into parallel tasks;
- a distributed engine is already part of the platform;
- required throughput justifies operational complexity.

## 44. Do not distribute every small task

For a 2 MB CSV file, a simple Python or SQL script may be faster and cheaper than starting a cluster.

Cluster startup, scheduling, network and coordination have overhead.

Use the simplest tool that meets:
- data size;
- runtime;
- reliability;
- latency;
- cost;
- maintainability requirements.

---

# Part 12 — Common mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Parallel and distributed mean the same thing | Parallel work can happen on one machine | Explain the machine boundary |
| More workers always means faster | Bottlenecks and overhead remain | Measure the workload |
| More partitions are always better | Tiny tasks create scheduling overhead | Tune based on data and engine |
| Ignore shuffle | Data movement may dominate runtime | Inspect execution stages |
| Ignore skew | One worker can dominate completion time | Inspect task distribution |
| Retry every task blindly | Side effects may duplicate | Design idempotent writes |
| Read output while job is incomplete | Partial results may be consumed | Publish only validated output |
| Use a cluster for tiny files | Startup and management overhead | Use a simpler process |
| Claim exactly-once casually | End-to-end semantics are complex | Define source, processing and sink guarantees |

---

# Part 13 — Practice questions

## Exercise 1

What is distributed computing?

**Answer:** Using multiple connected computers or workers to complete parts of a workload and coordinate results.

## Exercise 2

What is horizontal scaling?

**Answer:** Adding more machines or workers to increase capacity.

## Exercise 3

What is a partition?

**Answer:** A subset of a dataset processed as a unit.

## Exercise 4

What is shuffle?

**Answer:** Moving data between workers so related records can be processed together.

## Exercise 5

Why can a job with many workers still be slow?

**Answer:** It may be limited by shuffle, network, storage, skew, serial work or an external dependency.

## Exercise 6

What is data skew?

**Answer:** Uneven distribution of data across partitions or workers.

## Exercise 7

Why does idempotency matter?

**Answer:** A retried task should not create duplicate or incorrect side effects.

## Exercise 8

Should a tiny CSV always be processed with a distributed cluster?

**Answer:** No. A local tool may be simpler, faster and cheaper.

---

# Part 14 — Revision sheet

| Term | Remember |
|---|---|
| Distributed computing | Multiple connected machines coordinate work |
| Node | Participating computer |
| Worker | Component that performs assigned tasks |
| Cluster | Group of connected compute resources |
| Job | Larger submitted workload |
| Task | Unit of work |
| Sequential | One step after another |
| Parallel | Multiple tasks at once |
| Vertical scaling | Increase one machine's capacity |
| Horizontal scaling | Add more machines |
| Partition | Subset of data |
| Shuffle | Data movement between workers |
| Data locality | Process close to data where possible |
| Fault tolerance | Recover from component failure |
| Retry | Execute failed work again |
| Idempotency | Repeating an operation has the same defined effect |
| Data skew | Uneven data distribution |
| Straggler | Unusually slow task |
| Partial output | Incomplete result published before job completion |

## Final self-check

- [ ] Explain sequential, parallel and distributed processing.
- [ ] Explain node, worker, cluster, job and task.
- [ ] Compare vertical and horizontal scaling.
- [ ] Explain partitioning and shuffle.
- [ ] Explain data locality and network overhead.
- [ ] Explain retries, fault tolerance and idempotency.
- [ ] Identify data skew and stragglers.
- [ ] Explain why distributed processing is not needed for every small task.
- [ ] Run the local partition simulation and validate totals.

**Next lesson — Day 55: Apache Spark Fundamentals.** You will learn Spark's driver, executors, cluster manager, SparkSession and DataFrames, then read a dataset and inspect its structure.
