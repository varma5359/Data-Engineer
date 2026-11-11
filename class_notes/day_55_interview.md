# Day 55 Interview Guide — Apache Spark Fundamentals

## 1. What is Apache Spark?

Apache Spark is a data processing engine that supports batch processing, SQL/DataFrames, streaming and other workloads. It can run locally or across a cluster. It is commonly used to transform large datasets.

## 2. What is PySpark?

PySpark is the Python API for Apache Spark. It allows Python developers to create Spark sessions and work with Spark DataFrames while Spark performs the underlying execution.

## 3. Is Spark a database?

No. Spark is primarily a processing engine. It can read from and write to databases and storage systems, but it is not itself a relational database in the same sense as PostgreSQL.

## 4. What is the Spark driver?

The driver coordinates a Spark application. It creates the application context, builds execution plans, schedules work and tracks tasks.

## 5. What is an executor?

An executor is a process that runs tasks for a Spark application. It processes assigned partitions and can cache or persist data when configured.

## 6. What is a cluster manager?

A cluster manager allocates resources for Spark applications in a cluster. Examples include Spark Standalone, YARN and Kubernetes. The cluster manager allocates resources; the driver coordinates the application.

## 7. What is a SparkSession?

SparkSession is the main entry point for working with Spark DataFrames and SQL. It provides access to reading data, creating DataFrames, running SQL and configuring the Spark application.

## 8. What does local[*] mean?

It runs Spark locally using available local CPU cores. It is useful for learning and small tests but does not mean a multi-machine cluster has been created.

## 9. What is a Spark DataFrame?

A Spark DataFrame is a distributed collection of data organized into named columns. It has rows, columns, a schema, partitions and an execution plan.

## 10. What is a schema?

A schema defines the structure of a DataFrame, including column names, data types, nullability and nested fields. It helps the engine interpret data consistently.

## 11. What is schema inference?

Schema inference examines input data and estimates column types. It is convenient for exploration but can guess incorrectly or behave differently when data changes. Explicit schemas are more predictable for production pipelines.

## 12. What is the difference between show() and printSchema()?

show() displays sample rows. printSchema() displays the DataFrame's column structure and types. Neither replaces data quality validation.

## 13. What does count() do?

count() is an action that calculates the number of rows in a DataFrame. It may trigger Spark execution and scan the input data.

## 14. Why should you avoid collect() on a large DataFrame?

collect() brings all rows to the driver process. A large result can exhaust driver memory. Keep large datasets distributed and collect only small results when necessary.

## 15. What is the difference between pandas and Spark DataFrames?

Pandas DataFrames normally operate in one Python process and are constrained by local resources. Spark DataFrames are designed for distributed execution across workers. Spark has startup and scheduling overhead, so pandas may be simpler and faster for small data.

## 16. What is lazy evaluation?

Spark often builds a plan for transformations instead of immediately processing all rows. An action such as show(), count() or write() triggers the required execution. This gives Spark an opportunity to optimize the plan.

## 17. Why might Spark fail to start with a Java gateway error?

Possible causes include a missing or incompatible Java runtime, incorrect JAVA_HOME or a Spark/Java version mismatch. I would verify the Spark release's supported Java version and the active environment configuration.

## 18. Scenario: Spark reads a CSV but amount is inferred as integer. Later files contain decimals.

I would define an explicit decimal schema and validate malformed or incompatible values. I would not rely on inference for a financial field because type changes can affect downstream calculations.

## 19. Scenario: a developer calls collect() on a huge dataset and the driver crashes.

I would remove the full collection, keep the data distributed and use Spark transformations, aggregations or bounded samples. If the final result is large, write it to storage rather than moving all rows to the driver.

## 20. Explain your Day 55 project.

I installed PySpark in a local environment, created a SparkSession in local mode, read a sample orders CSV, inspected rows and schema, checked columns and count, and compared inferred schema with an explicit decimal schema. I would state clearly that this was local practice, not a multi-node cluster benchmark.

## Common interview traps

- Saying PySpark and Spark are separate engines.
- Calling Spark a database.
- Claiming local[*] is a cluster.
- Confusing driver and executor.
- Trusting inferred schema without validation.
- Collecting all rows to the driver.
- Assuming Spark is faster than pandas for every dataset.
- Ignoring Java compatibility.

## Readiness checklist

- [ ] Explain Spark and PySpark.
- [ ] Explain driver, executor and cluster manager.
- [ ] Explain SparkSession and DataFrames.
- [ ] Read and inspect a CSV.
- [ ] Explain schema inference and explicit schema.
- [ ] Explain lazy evaluation and collect risks.
- [ ] Defend local versus cluster mode.
