# Day 54 Interview Guide — Distributed Computing

## 1. What is distributed computing?

Distributed computing uses multiple connected computers or workers to complete parts of a workload and coordinate results. It allows a system to process larger workloads than one machine may handle efficiently, but it introduces network, coordination and failure-handling concerns.

## 2. What is the difference between sequential, parallel and distributed processing?

Sequential processing performs work one step after another. Parallel processing runs multiple tasks at the same time, possibly on one machine with multiple CPU cores. Distributed processing coordinates work across multiple networked machines. A distributed system can also use parallelism within each machine.

## 3. What is a node?

A node is a computer or machine participating in a distributed system. It contributes resources such as CPU, memory, storage and network connectivity.

## 4. What is a worker?

A worker is a process or compute component that performs assigned tasks. Depending on the platform, workers may run on cluster nodes or inside containers.

## 5. What is a cluster?

A cluster is a group of connected compute resources managed to work together. A data processing cluster may include a coordinator, workers, a cluster manager and storage connectivity.

## 6. What is horizontal scaling?

Horizontal scaling adds more machines or workers to increase capacity. It can improve throughput when work can be divided, but it also adds coordination, network and operational overhead.

## 7. What is vertical scaling?

Vertical scaling increases resources on one machine, such as CPU or memory. It can be simpler but is limited by machine sizes and may create a large single-machine dependency.

## 8. What is a partition?

A partition is a subset of a dataset processed as a unit. Multiple partitions allow independent tasks to run in parallel. Partition size and count affect scheduling, memory and output behavior.

## 9. What is shuffle?

Shuffle is data movement across workers so records that need to be processed together can be brought to the same destination. Grouping, joins and sorting can require shuffle. It can be expensive because it uses network, memory and disk.

## 10. What is data locality?

Data locality means processing data close to where it is stored when the platform supports it. It can reduce unnecessary network transfer. Cloud object storage and compute placement should be considered together.

## 11. What is fault tolerance?

Fault tolerance is the ability to continue operating or recover when a component fails. Distributed engines may retry failed tasks or reschedule them on other workers. The pipeline must still protect against duplicate side effects and partial output.

## 12. Why is idempotency important?

A task may be executed more than once because of retries. An idempotent operation has the same defined final effect when repeated. Idempotent writes and stable run identifiers help prevent duplicate records or incorrect counters.

## 13. What is data skew?

Data skew is an uneven distribution of records across partitions or workers. A worker with far more data can become the bottleneck. Engineers inspect partition sizes and task durations and may use better keys, pre-aggregation or skew-aware engine features.

## 14. What is a straggler?

A straggler is a task that takes much longer than other tasks in the same stage. Causes include skew, slow storage, network issues and resource contention. The overall job may wait for the straggler to finish.

## 15. Does adding workers always make processing faster?

No. The workload may be limited by serial code, external APIs, storage, shuffle, network, skew or scheduling overhead. More workers can also increase cost. Measure the bottleneck before scaling.

## 16. Why not create thousands of partitions?

Too many tiny partitions can create task scheduling, metadata and file overhead. The right number depends on data size, engine behavior, worker capacity and downstream needs.

## 17. What is the difference between high availability and fault tolerance?

High availability focuses on keeping a service available with minimal interruption. Fault tolerance is the broader ability to continue or recover when components fail. The exact terms can overlap, but they are not a substitute for backups and correctness design.

## 18. What is the exactly-once processing challenge?

A processing engine may retry work, but an external sink can still receive duplicate effects unless the sink and write protocol support the required guarantees. Engineers must consider source delivery, checkpoints, transactions, idempotency and deduplication end to end.

## 19. Scenario: one worker takes ten times longer than the others.

I would inspect partition sizes, key distribution, task input, network and disk metrics. If the task has much more data, skew is likely. I would consider a better partition key, skew-aware processing or pre-aggregation, then validate that the business result remains unchanged.

## 20. Scenario: a job fails after writing some output files.

I would prevent downstream consumers from reading the incomplete output. A safer pattern is to write to a run-specific temporary location, validate the result and publish it only after successful completion using a platform-appropriate commit mechanism. I would also clean up abandoned output safely.

## 21. When should you avoid distributed processing?

For small datasets or simple jobs, a local Python or SQL process may be faster, cheaper and easier to maintain. Distributed processing is justified when data size, runtime, throughput or reliability needs outweigh cluster startup and coordination overhead.

## 22. Explain your Day 54 experiment.

I split a small sales list into three partitions, calculated partial totals and combined them. I compared the result with the original total and intentionally demonstrated how a missing partition range creates an incorrect result. I then designed a worker assignment and documented shuffle, skew, retry safety and scaling trade-offs. The Python simulation ran locally; it was not a distributed cluster benchmark.

## Common interview traps

- Saying parallel and distributed are identical.
- Assuming more workers always improve speed.
- Ignoring shuffle and network transfer.
- Ignoring data skew.
- Retrying tasks without considering duplicate side effects.
- Publishing partial output.
- Using distributed compute for tiny datasets without justification.
- Claiming exactly-once without defining the system boundary.

## Readiness checklist

- [ ] Explain nodes, workers, clusters, jobs and tasks.
- [ ] Compare sequential, parallel and distributed processing.
- [ ] Explain horizontal and vertical scaling.
- [ ] Explain partitions and shuffle.
- [ ] Explain fault tolerance and idempotency.
- [ ] Identify skew and stragglers.
- [ ] Defend when distributed processing is appropriate.
