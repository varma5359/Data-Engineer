# Day 59 Interview Guide — Cloud and Distributed Pipeline Integration

## 1. What is an end-to-end data pipeline?

It is a sequence of source extraction or event ingestion, raw landing, validation, transformation, curated publication, monitoring and recovery. A production pipeline includes security, ownership, repeatability and correctness, not only transformation code.

## 2. Why separate ingestion and transformation?

Ingestion receives data and preserves the source copy. Transformation reads stored data and applies business rules. Separating them supports replay and debugging without repeatedly extracting from the source.

## 3. What is a raw zone?

A raw zone preserves source data as received, subject to access and retention controls. It supports reprocessing, audit and comparison with transformed data.

## 4. What is a curated zone?

A curated zone contains business-ready datasets with documented grain, schema, ownership, quality checks and freshness expectations.

## 5. What is idempotency in a data pipeline?

An operation is idempotent when repeating it has the same intended final effect under defined conditions. It helps prevent duplicate output when jobs retry. Examples include safe partition replacement, merge/upsert by a business key or a transactional table commit.

## 6. Why use run IDs?

A run ID identifies a pipeline execution. It supports traceability, audit, retry analysis, output isolation and comparison of input and output for a specific run.

## 7. Why should a pipeline publish output only after validation?

If consumers read output while a job is still writing, they may see incomplete data. A safer pattern writes to a temporary or run-specific location, validates the output and then publishes it using a platform-appropriate commit mechanism.

## 8. What is a checkpoint?

A checkpoint is persisted progress or state used by a processing system to recover. Streaming checkpoints can track progress and state. Batch workflows may use run ledgers or durable intermediate outputs. A checkpoint is not the same as a backup.

## 9. What is the difference between batch and streaming?

Batch processes bounded groups of records on a schedule or trigger. Streaming processes continuous events or small increments as they arrive. Streaming is useful when low latency is required but introduces state, checkpoint, late-data and operational complexity.

## 10. What is micro-batch processing?

Micro-batch processing collects incoming records over short intervals and processes them as small batches. Spark Structured Streaming commonly uses micro-batch execution by default for supported queries.

## 11. What is Apache Kafka?

Kafka is a distributed event streaming platform used to publish, store and consume event streams. Producers write records to topics and consumers read them. Kafka is not the same as object storage.

## 12. What is a Kafka broker?

A broker is a Kafka server that stores and serves topic partitions. A Kafka cluster usually contains multiple brokers.

## 13. What is a Kafka topic?

A topic is a named stream of records, such as orders or payments. Producers publish records to topics and consumers subscribe to them.

## 14. What is a Kafka partition?

A partition is a subdivision of a topic. Partitions enable parallelism and distribute data across brokers. Ordering is guaranteed within a partition, not globally across all partitions of a topic.

## 15. What is a Kafka producer?

A producer is an application that publishes records to Kafka topics. An order service may publish an order-created event.

## 16. What is a Kafka consumer?

A consumer is an application that reads records from Kafka. A Spark streaming application can consume order events and transform them.

## 17. What is a consumer group?

A consumer group is a set of consumers that cooperate to read a topic. Partitions are assigned among consumers in the group. Multiple groups can independently consume the same topic for different applications.

## 18. What is an offset?

An offset identifies a record's position within a Kafka partition. Consumers track offsets to manage progress. Offsets are partition-specific.

## 19. What is consumer lag?

Consumer lag is the gap between the latest available records and the consumer's progress. It can indicate that processing is slower than event arrival or that a consumer is unhealthy.

## 20. How can ordering be maintained for events belonging to one order?

A producer can use a stable key such as order_id so related events are routed consistently to the same partition. Kafka preserves ordering within that partition. There is no global order across all topic partitions.

## 21. How would Spark read from S3?

A Spark deployment needs compatible S3A/Hadoop connector dependencies, an approved AWS identity such as a workload role, network connectivity, correct bucket/region configuration and any required encryption-key permissions. I would verify Spark, Hadoop and connector compatibility and avoid hardcoded access keys.

## 22. Scenario: a pipeline rerun duplicates yesterday's orders.

I would inspect whether the output uses append mode, whether source event IDs are unique, whether the previous run partially committed and whether run status was recorded. I would implement idempotent writes using a stable key, controlled partition replacement or a supported transactional table mechanism.

## 23. Scenario: the curated folder contains only some output files after a failure.

I would prevent consumers from reading the partial output, isolate the failed run, validate or remove temporary output safely and rerun using an idempotent strategy. I would publish only after all validation checks succeed.

## 24. Scenario: Kafka consumer lag keeps increasing.

I would compare producer throughput with consumer processing throughput, inspect errors and retries, task duration, sink latency and partition count. I would identify whether processing, network or downstream storage is the bottleneck before adding consumers or partitions.

## 25. How do you choose batch or streaming?

I start with the business latency requirement. If a daily report by 7:00 AM is sufficient, scheduled batch may be simpler and cheaper. If the business needs decisions within seconds or minutes of events, streaming or micro-batch may be justified. I also compare operational skills, state, replay, correctness and cost.

## 26. Explain your Day 59 project.

I designed a batch architecture from source systems through raw storage, Spark processing, quality checks and curated publication. I ran a local Spark simulation, validated required columns and amount totals, wrote and read back Parquet outputs, and documented idempotency, monitoring and security. I also compared batch and streaming and explained Kafka's brokers, topics, partitions, producers, consumers, groups and offsets. I would distinguish the local simulation from a real S3/Kafka deployment.

## Common interview traps

- Calling a Spark script a complete production pipeline.
- Publishing incomplete output.
- Appending duplicate records on every retry.
- Claiming local paths prove S3 integration.
- Choosing streaming without a latency requirement.
- Saying Kafka guarantees global topic ordering.
- Confusing topic, partition, broker and offset.
- Claiming exactly-once without defining source and sink behavior.
- Ignoring consumer lag, security and cost.

## Readiness checklist

- [ ] Draw an end-to-end cloud batch pipeline.
- [ ] Explain raw, processed and curated layers.
- [ ] Explain run IDs and idempotency.
- [ ] Explain safe output publication and checkpoints.
- [ ] Compare batch, micro-batch and streaming.
- [ ] Explain Kafka brokers, topics and partitions.
- [ ] Explain producers, consumers, groups and offsets.
- [ ] Describe Spark-to-S3 integration requirements.
- [ ] Explain monitoring, security and recovery.
