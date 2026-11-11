# Day 55: Apache Spark Fundamentals

**Phase:** 6 — Cloud and Distributed Processing  
**Learning time:** 2–3 hours  
**Audience:** Complete beginners and junior Data Engineer candidates  
**Technology:** Apache Spark with PySpark  
**Topic boundary:** Class notes only. Project workbook and interview guide are separate files.

> Day 54 explained distributed computing. Today we meet Apache Spark, a processing engine used to work with large datasets. You will understand the driver, executors, cluster manager, SparkSession and DataFrames. Then you will create a local Spark session, read a CSV file and inspect the dataset. Local mode is for learning; it is not proof that a distributed cluster is running.

## Learning outcomes

- Explain Apache Spark.
- Understand Spark's role in data engineering.
- Distinguish driver, executor and cluster manager.
- Explain local mode versus cluster mode.
- Understand SparkSession.
- Understand DataFrames and schemas.
- Read CSV data with PySpark.
- Inspect rows, columns, schema and summary information.
- Understand lazy evaluation at a beginner level.
- Recognize common Spark setup and data-reading mistakes.

---

# Part 1 — Why Apache Spark?

## 1. A company data problem

TechStore receives millions of order and clickstream records.

The data engineering team needs to:
- read files;
- filter invalid records;
- join datasets;
- aggregate sales;
- write transformed data.

A normal Python script may be enough for small data. As data grows, a single computer may not provide enough memory or processing capacity.

Apache Spark provides a distributed data processing engine that can execute work across multiple machines when configured as a cluster.

## 2. What is Apache Spark?

**Beginner definition:** Apache Spark is an engine for processing data. It provides APIs for working with structured data and other workloads, and can run locally or across a cluster.

Spark supports:
- batch data processing;
- SQL and DataFrame operations;
- streaming workloads;
- machine learning libraries;
- graph processing libraries.

For this course, the main focus is Spark DataFrames for batch data engineering.

## 3. What is PySpark?

PySpark is the Python API for Apache Spark.

It allows Python developers to create Spark sessions and use Spark DataFrame operations.

Remember:
- Python is the language used to write the code;
- Spark is the processing engine;
- PySpark connects Python code to Spark functionality.

## 4. Spark is not a database

Spark can read from and write to many storage systems, but it is primarily a processing engine.

| Technology | Main purpose |
|---|---|
| PostgreSQL | Relational database |
| Amazon S3 | Object storage |
| Apache Spark | Distributed data processing engine |
| Parquet | Columnar file format |
| Kafka | Event streaming platform |

A Spark job may read Parquet files from object storage, transform them and write Parquet output.

---

# Part 2 — Spark architecture

## 5. Main architecture components

A Spark application commonly involves:

1. Driver
2. Cluster manager
3. Executors
4. Tasks
5. Data partitions

## 6. What is the driver?

The driver is the process that coordinates a Spark application.

It:
- creates the Spark application context;
- builds the execution plan from user operations;
- coordinates jobs and tasks;
- tracks execution;
- collects results when requested.

The driver is not simply another name for an executor.

## 7. What is an executor?

An executor is a process that runs tasks for a Spark application.

Executors:
- process assigned partitions;
- perform transformations;
- cache or persist data when configured;
- report status and results to the driver.

Executors run on worker resources in cluster deployments. In local mode, execution happens within the local environment rather than a separate multi-machine cluster.

## 8. What is a cluster manager?

A cluster manager allocates resources for applications in a cluster.

Spark can run with different cluster management options, such as:
- Standalone cluster manager;
- Hadoop YARN;
- Kubernetes;
- managed cluster services.

The cluster manager is responsible for resource allocation, while Spark's driver coordinates the application.

## 9. Understand the flow

~~~text
Python application
       |
       v
Spark Driver
       |
       v
Cluster Manager requests resources
       |
       v
Executors are started
       |
       v
Tasks process data partitions
       |
       v
Results and status return to driver
~~~

This is a conceptual diagram. Exact deployment details depend on the cluster manager and Spark mode.

## 10. What is a task?

A task is a unit of work executed on a partition.

A Spark job may be divided into stages, and stages contain tasks.

You will learn more about stages and execution plans in the performance lessons.

## 11. Local mode versus cluster mode

| Mode | Meaning | Typical use |
|---|---|---|
| Local mode | Spark runs on the current machine | Learning, testing, small local work |
| Cluster mode | Spark application uses cluster resources | Larger shared or production workloads |

A local Spark session does not prove that you have configured multiple workers or a cloud cluster.

---

# Part 3 — SparkSession

## 12. What is SparkSession?

SparkSession is the main entry point for working with Spark DataFrames and SQL in modern PySpark applications.

Through SparkSession you can:
- read data;
- create DataFrames;
- run Spark SQL;
- access configuration;
- manage the Spark application session.

## 13. Create a local Spark session

First, install PySpark in your active Python environment:

~~~bash
python -m pip install pyspark
~~~

Spark also requires a compatible Java runtime. The required Java version depends on the Spark version you install. Check the official compatibility documentation for the selected Spark release.

Verify Python and PySpark:

~~~bash
python --version
python -m pip show pyspark
~~~

Create a session:

~~~python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("TechStoreDataExploration")
    .master("local[*]")
    .getOrCreate()
)

print("Spark session created")
print("Spark version:", spark.version)
~~~

Example output:

~~~text
Spark session created
Spark version: 3.x.x
~~~

The exact version will be the version installed in your environment.

### Understand each line

- The import makes SparkSession available.
- builder starts session configuration.
- appName gives the application a readable name.
- master("local[*]") requests local execution using available local cores.
- getOrCreate() returns an existing session or creates one.
- spark.version displays the installed Spark version.

Do not use local[*] to claim that a distributed cluster has been created. It is local execution.

## 14. Why getOrCreate?

A notebook may execute cells multiple times.

getOrCreate() can reuse an existing SparkSession instead of always creating another session.

In a long-running application, manage the session lifecycle deliberately.

## 15. Stop a Spark session

At the end of a script:

~~~python
spark.stop()
print("Spark session stopped")
~~~

Expected output:

~~~text
Spark session stopped
~~~

In an interactive notebook, do not stop the session after every cell if you still need it for later cells.

---

# Part 4 — Understand DataFrames

## 16. What is a DataFrame?

A Spark DataFrame is a distributed collection of data organized into named columns.

It is similar in appearance to a relational table or pandas DataFrame, but Spark DataFrames are designed for Spark's execution engine.

A DataFrame has:
- rows;
- columns;
- a schema;
- partitions;
- an execution plan.

## 17. Example dataset

| order_id | customer_id | city | amount |
|---:|---|---|---:|
| 101 | C001 | Hyderabad | 250 |
| 102 | C002 | Chennai | 499 |
| 103 | C001 | Hyderabad | 100 |
| 104 | C003 | Bengaluru | 750 |

A Spark DataFrame can represent this data.

## 18. What is a schema?

A schema describes the structure of a DataFrame.

It includes:
- column names;
- data types;
- whether columns allow null values;
- nested structure where applicable.

Example schema:

| Column | Type |
|---|---|
| order_id | integer |
| customer_id | string |
| city | string |
| amount | decimal or numeric |

A schema helps Spark interpret data and detect incompatible values.

## 19. Why explicit schema is useful

Spark can infer a schema from data, but inference can:
- take additional work;
- guess an unintended type;
- behave differently when values vary;
- cause inconsistent results between files.

For important production datasets, define and validate the expected schema.

---

# Part 5 — Create and read a CSV DataFrame

## 20. Create the practice folder

Create a folder:

~~~text
spark_day55/
~~~

Inside it, create a file named orders.csv.

~~~csv
order_id,customer_id,city,amount
101,C001,Hyderabad,250
102,C002,Chennai,499
103,C001,Hyderabad,100
104,C003,Bengaluru,750
105,C004,Chennai,300
~~~

Save the file.

## 21. Start Spark and read CSV

Run the following code in a Python file or notebook. Run the cells in order.

~~~python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("Day55SparkExploration")
    .master("local[*]")
    .getOrCreate()
)

orders = spark.read.csv(
    "spark_day55/orders.csv",
    header=True,
    inferSchema=True
)

print("CSV loaded")
~~~

Expected output:

~~~text
CSV loaded
~~~

The file path is relative to the current working directory. If Spark reports that the path does not exist, check where the Python process is running and use the correct path.

## 22. Understand the read options

| Option | Meaning |
|---|---|
| header=True | First row contains column names |
| inferSchema=True | Spark examines data to infer types |
| path | File or directory to read |

Do not assume inferSchema is always the best production choice. Explicit schema is usually more predictable for governed pipelines.

## 23. Display the rows

~~~python
orders.show()
~~~

Expected output:

~~~text
+--------+-----------+---------+------+
|order_id|customer_id|     city|amount|
+--------+-----------+---------+------+
|     101|       C001|Hyderabad|   250|
|     102|       C002|  Chennai|   499|
|     103|       C001|Hyderabad|   100|
|     104|       C003|Bengaluru|   750|
|     105|       C004|  Chennai|   300|
+--------+-----------+---------+------+
~~~

The spacing may vary slightly by Spark version.

## 24. Understand show()

show() displays rows from a DataFrame in a readable table format.

By default, it displays a limited number of rows.

It is useful for inspection, but do not use show() to print millions of rows.

## 25. Inspect the schema

~~~python
orders.printSchema()
~~~

Expected output (inferred types may vary if the source values change):

~~~text
root
 |-- order_id: integer (nullable = true)
 |-- customer_id: string (nullable = true)
 |-- city: string (nullable = true)
 |-- amount: integer (nullable = true)
~~~

Why is amount inferred as integer? Every amount in the sample file is a whole number.

If a future file contains decimal amounts, inference may produce a different type. Production pipelines should define the intended numeric type.

## 26. Get column names

~~~python
print(orders.columns)
~~~

Expected output:

~~~text
['order_id', 'customer_id', 'city', 'amount']
~~~

## 27. Get the DataFrame data types

~~~python
print(orders.dtypes)
~~~

Example output:

~~~text
[('order_id', 'int'), ('customer_id', 'string'), ('city', 'string'), ('amount', 'int')]
~~~

## 28. Count rows

~~~python
print("Number of rows:", orders.count())
~~~

Expected output:

~~~text
Number of rows: 5
~~~

count() is an action. It asks Spark to calculate the number of rows.

## 29. Count columns

~~~python
print("Number of columns:", len(orders.columns))
~~~

Expected output:

~~~text
Number of columns: 4
~~~

len(orders.columns) evaluates the local list of column names. It does not scan every data row.

## 30. Describe numeric columns

~~~python
orders.describe("amount").show()
~~~

Expected summary values:

| summary | amount |
|---|---:|
| count | 5 |
| mean | 379.8 |
| stddev | approximately 252.7 |
| min | 100 |
| max | 750 |

Spark's displayed precision may differ. The summary is calculated from the data.

---

# Part 6 — Use an explicit schema

## 31. Why define schema?

Suppose amount represents money.

A whole-number type may not be sufficient if cents are present.

For financial data, a decimal type is often preferable to binary floating point.

## 32. Define a schema

~~~python
from pyspark.sql.types import (
    StructType,
    StructField,
    IntegerType,
    StringType,
    DecimalType
)

orders_schema = StructType([
    StructField("order_id", IntegerType(), True),
    StructField("customer_id", StringType(), True),
    StructField("city", StringType(), True),
    StructField("amount", DecimalType(10, 2), True)
])
~~~

Understand:
- StructType describes the whole row structure;
- StructField defines one column;
- IntegerType represents an integer;
- StringType represents text;
- DecimalType(10, 2) supports precision 10 with 2 decimal places;
- True means the field is nullable.

## 33. Read with the explicit schema

~~~python
orders_decimal = spark.read.csv(
    "spark_day55/orders.csv",
    header=True,
    schema=orders_schema
)

orders_decimal.printSchema()
orders_decimal.show()
~~~

Expected schema includes:

~~~text
 |-- amount: decimal(10,2) (nullable = true)
~~~

The file values are parsed according to the schema. Invalid values may become null or cause parsing behavior depending on read options and Spark version. Validate the result.

## 34. Why nullable matters

Nullable means the schema allows a value to be null.

It does not mean every row actually contains null.

Business rules may require a column to be non-null even if the file reader allows nulls. Data quality checks should enforce business expectations.

---

# Part 7 — Spark DataFrame versus pandas DataFrame

## 35. Compare them

| Area | pandas DataFrame | Spark DataFrame |
|---|---|---|
| Main execution | Local Python process | Spark engine |
| Data scale | Usually constrained by local memory | Can distribute across workers |
| Execution | Many operations execute eagerly | Transformations are generally lazy |
| Parallel execution | Not a distributed cluster by default | Designed for distributed execution |
| Typical use | Local analysis and small/medium data | Large-scale processing and pipelines |

Spark is not automatically faster for every dataset. For small data, pandas may be simpler and faster because Spark has startup and scheduling overhead.

## 36. Do not collect large data to the driver

Operations such as collect() bring results to the driver process.

For a tiny training dataset, this may be fine.

For a large dataset, collecting every row can exhaust driver memory.

Use distributed operations and collect only small results when necessary.

---

# Part 8 — Lazy evaluation introduction

## 37. What does lazy evaluation mean?

Many Spark DataFrame transformations do not immediately process all data.

Spark builds a logical plan describing the work.

An action asks Spark to execute the required plan.

Examples of transformations:
- select;
- filter;
- withColumn;
- groupBy.

Examples of actions:
- show;
- count;
- collect;
- write.

You will study these in detail on Day 56.

## 38. Why lazy evaluation helps

Spark can inspect and optimize a sequence of operations before execution.

It can avoid doing work until a result is requested.

But lazy evaluation can surprise beginners: a line that creates a filtered DataFrame may not read every row immediately.

---

# Part 9 — Common setup and data errors

## 39. Error: Java gateway process exited

Possible causes:
- incompatible or missing Java runtime;
- unsupported Spark/Java combination;
- incorrect JAVA_HOME;
- environment configuration issue.

Check the Spark version's Java compatibility documentation and verify the active Java runtime.

## 40. Error: path does not exist

Possible causes:
- wrong current working directory;
- spelling mismatch;
- file saved in another folder;
- relative path used incorrectly.

Print the current working directory with Python's os.getcwd() and verify the file location.

## 41. Error: cannot infer schema

Possible causes:
- empty file;
- missing header or malformed content;
- no rows from which to infer types.

Provide an explicit schema and validate the source file.

## 42. Error: wrong data type

Possible causes:
- inference guessed a type from sample values;
- source values are inconsistent;
- decimal values appear in a field previously containing integers.

Define the expected schema and validate invalid values.

## 43. Error: too many rows displayed

Use show(n) with a small n or select only required columns. Do not print full production datasets.

---

# Part 10 — Common beginner mistakes

| Mistake | Why it is wrong | Better practice |
|---|---|---|
| Spark is a database | It is a processing engine | Explain its role |
| PySpark is the engine itself | It is the Python API | Separate Python API and Spark engine |
| local[*] means cloud cluster | It runs locally | Identify execution mode honestly |
| Driver and executor are the same | They have different roles | Learn coordinator versus task execution |
| inferSchema is always safe | Types can be inferred incorrectly | Define schema for governed data |
| show() prints all data | It shows a limited sample | Use deliberate inspection |
| collect() is safe for all data | Driver memory can fail | Keep large data distributed |
| Spark always beats pandas | Startup/coordination overhead exists | Match tool to workload |
| Spark read succeeded means data is valid | Parse errors and nulls may remain | Validate schema and values |

---

# Part 11 — Practice questions

## Exercise 1

What is Apache Spark?

**Answer:** A data processing engine that can execute workloads locally or across a cluster.

## Exercise 2

What is PySpark?

**Answer:** The Python API for using Spark.

## Exercise 3

What does the driver do?

**Answer:** Coordinates the Spark application and schedules work.

## Exercise 4

What does an executor do?

**Answer:** Runs tasks assigned by the Spark application.

## Exercise 5

What does local[*] mean?

**Answer:** Run Spark locally using available local cores. It does not create a multi-machine cluster.

## Exercise 6

What is a DataFrame?

**Answer:** A distributed collection of data organized into named columns.

## Exercise 7

Why use explicit schema?

**Answer:** It makes expected column types clear and avoids unreliable inference.

## Exercise 8

Why avoid collect() on large datasets?

**Answer:** It transfers data to the driver and can exhaust driver memory.

---

# Part 12 — Revision sheet

| Term | Remember |
|---|---|
| Apache Spark | Data processing engine |
| PySpark | Python API for Spark |
| Driver | Coordinates the application |
| Executor | Runs assigned tasks |
| Cluster manager | Allocates cluster resources |
| Task | Unit of work |
| SparkSession | Main entry point for DataFrame/SQL work |
| DataFrame | Distributed rows and named columns |
| Schema | Column names and data types |
| inferSchema | Spark estimates types from input |
| Explicit schema | Engineer defines expected structure |
| show() | Displays sample rows |
| printSchema() | Displays schema |
| count() | Counts rows |
| collect() | Brings results to driver |
| Local mode | Spark runs on the current machine |
| Lazy evaluation | Work is planned and run when an action requires it |

## Final self-check

- [ ] Explain Spark and PySpark.
- [ ] Explain driver, executor and cluster manager.
- [ ] Compare local and cluster mode.
- [ ] Create a SparkSession.
- [ ] Read a CSV into a DataFrame.
- [ ] Use show, printSchema, columns, dtypes and count.
- [ ] Explain schema inference and explicit schema.
- [ ] Explain lazy evaluation at a beginner level.
- [ ] Explain why collect() can be dangerous.
- [ ] Troubleshoot basic Java and file path errors.

**Next lesson — Day 56: Spark Transformations and Actions.** You will learn select, filter, withColumn, groupBy, aggregations and why transformations are lazy until an action executes the plan.
