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


---

# Part 13 — Prepare the Windows practice environment carefully

Spark setup is sometimes harder than the first DataFrame lesson. Separate the requirements and verify them one by one.

## 44. Understand the three main components

For local PySpark practice, you need:

1. Python — runs your PySpark application code.
2. PySpark — provides the Python API and Spark runtime components.
3. A compatible Java runtime — Spark uses the Java Virtual Machine.

Installing Python alone does not install Spark. Installing PySpark does not guarantee that Java is configured correctly.

## 45. Check Python

Open CMD:

~~~bat
python --version
~~~

Example output:

~~~text
Python 3.x.x
~~~

The exact version depends on your installation. Use a Python version supported by the Spark release you choose.

If Python is not recognized, verify installation and PATH before continuing.

## 46. Check pip

~~~bat
python -m pip --version
~~~

This checks pip through the Python interpreter selected by the command. Using python -m pip helps avoid installing a package into a different Python installation.

## 47. Check Java

~~~bat
java -version
~~~

The command prints the active Java runtime version.

If Java is not recognized:
1. install a Java runtime supported by your chosen Spark release;
2. follow the official installation instructions;
3. configure PATH and JAVA_HOME if required;
4. close and reopen CMD;
5. run java -version again.

Do not assume one Java version works with every Spark release. Check the official Spark compatibility documentation for the exact version you install.

## 48. Install PySpark

~~~bat
python -m pip install pyspark
~~~

Expected behavior: pip downloads and installs PySpark and its dependencies, or reports that the package is already installed.

Check the installed package:

~~~bat
python -m pip show pyspark
~~~

The output includes the installed version and package location.

## 49. Confirm which Python is active

~~~bat
where python
~~~

Windows may show multiple Python executables. The first matching executable is commonly the one selected by CMD.

If Jupyter uses a different kernel, it may not see the same PySpark installation. Check the notebook kernel's Python environment.

## 50. Create the practice folder

In CMD:

~~~bat
mkdir spark_day55
cd spark_day55
mkdir data
~~~

Create the file:

~~~text
spark_day55/
    data/
        orders.csv
    day55_spark_practice.py
~~~

Save the notebook or Python file in the project folder. Keep the data file in the data folder.

## 51. Create the CSV

Create data/orders.csv with:

~~~csv
order_id,customer_id,city,amount
101,C001,Hyderabad,250.00
102,C002,Chennai,499.50
103,C001,Hyderabad,100.00
104,C003,Bengaluru,750.25
105,C004,Chennai,300.00
~~~

Save and close the file.

This dataset deliberately includes decimal amounts so the schema exercise can demonstrate money values.

Check that the file exists:

~~~bat
dir data
~~~

If Windows saved it as orders.csv.txt, turn on file-name extensions in File Explorer and correct the name.

---

# Part 14 — Create your first Spark application step by step

## 52. Cell 1 — Import SparkSession

~~~python
from pyspark.sql import SparkSession
~~~

**Goal:** Import the object used to create and manage a Spark application session.

**Expected output:** No printed output. A successful import normally produces no message.

If you see ModuleNotFoundError, check that PySpark is installed in the active Python environment.

## 53. Cell 2 — Create the SparkSession

~~~python
spark = (
    SparkSession.builder
    .appName("TechStoreDay55")
    .master("local[*]")
    .getOrCreate()
)

print("Spark is ready")
~~~

Expected output:

~~~text
Spark is ready
~~~

Understand each line:
- SparkSession.builder begins session configuration.
- appName gives the application a recognizable name.
- master("local[*]") requests local execution using available cores.
- getOrCreate() reuses an existing session or creates one.
- spark stores the session object.

This is local mode. It is not a cloud cluster.

## 54. Cell 3 — Inspect Spark version and application mode

~~~python
print("Spark version:", spark.version)
print("Application name:", spark.sparkContext.appName)
print("Master:", spark.sparkContext.master)
~~~

Example output:

~~~text
Spark version: 3.x.x
Application name: TechStoreDay55
Master: local[*]
~~~

The version number depends on your installation. The master value should show local execution for this exercise.

## 55. Cell 4 — Check the working directory

~~~python
import os

print(os.getcwd())
~~~

Expected output: the full folder path from which the Python process is running.

Why does this matter? Relative paths are resolved from the current working directory. If Spark cannot find data/orders.csv, first confirm this location.

## 56. Cell 5 — Read the CSV

~~~python
orders = spark.read.csv(
    "data/orders.csv",
    header=True,
    inferSchema=True
)

print("CSV DataFrame created")
~~~

Expected output:

~~~text
CSV DataFrame created
~~~

The DataFrame variable now describes the input. Spark may defer some processing until an action is requested.

## 57. Cell 6 — Display a small sample

~~~python
orders.show(5, truncate=False)
~~~

Expected rows:

| order_id | customer_id | city | amount |
|---:|---|---|---:|
| 101 | C001 | Hyderabad | 250.00 |
| 102 | C002 | Chennai | 499.50 |
| 103 | C001 | Hyderabad | 100.00 |
| 104 | C003 | Bengaluru | 750.25 |
| 105 | C004 | Chennai | 300.00 |

Spark's console table formatting may differ. The values should match the source CSV.

The argument 5 asks Spark to display up to five rows. truncate=False prevents long values from being shortened.

## 58. Cell 7 — Inspect the schema

~~~python
orders.printSchema()
~~~

Expected schema will include:
- order_id as an integer;
- customer_id as a string;
- city as a string;
- amount as a decimal or numeric type inferred from the file.

Exact inferred types can vary with Spark version and data. Inspect your own output.

## 59. Cell 8 — Inspect columns and data types

~~~python
print("Columns:", orders.columns)
print("Data types:", orders.dtypes)
~~~

Expected column names:

~~~text
['order_id', 'customer_id', 'city', 'amount']
~~~

The data type list is produced from the schema. Do not assume the inferred type is the business-approved type.

## 60. Cell 9 — Count rows

~~~python
row_count = orders.count()

print("Rows:", row_count)
~~~

Expected output:

~~~text
Rows: 5
~~~

count() is an action. Spark must execute enough of the plan to calculate the row count.

## 61. Cell 10 — Count columns

~~~python
column_count = len(orders.columns)

print("Columns:", column_count)
~~~

Expected output:

~~~text
Columns: 4
~~~

The columns list is local metadata. This does not scan all data rows.

## 62. Cell 11 — Inspect summary statistics

~~~python
orders.describe("amount").show()
~~~

Expected summary:
- count = 5;
- mean = 380.0;
- minimum = 100.0;
- maximum = 750.25.

Standard deviation is calculated by Spark. Displayed decimal precision may vary.

Check the mean manually:

~~~text
250.00 + 499.50 + 100.00 + 750.25 + 300.00 = 1899.75
1899.75 / 5 = 379.95
~~~

Therefore the expected mean is 379.95, subject to display rounding. Use the actual output and this arithmetic to verify it.

---

# Part 15 — Use an explicit schema for predictable data types

## 63. Why inference can be risky

Imagine the first file contains only whole-number amounts. Spark may infer an integer type.

A later file may contain 499.50. The schema can then differ or parsing may behave differently depending on how files are read.

For money, define the intended decimal type explicitly.

## 64. Import schema types

~~~python
from pyspark.sql.types import (
    StructType,
    StructField,
    IntegerType,
    StringType,
    DecimalType
)
~~~

This imports the schema building blocks.

## 65. Define the expected schema

~~~python
orders_schema = StructType([
    StructField("order_id", IntegerType(), False),
    StructField("customer_id", StringType(), False),
    StructField("city", StringType(), True),
    StructField("amount", DecimalType(12, 2), False)
])

print(orders_schema)
~~~

Expected output: Spark displays a schema definition containing the four named fields and their types.

Understand:
- StructType describes the row;
- StructField defines one column;
- IntegerType is a whole-number type;
- StringType stores text;
- DecimalType(12, 2) supports up to 12 total digits with 2 decimal places;
- False marks a field as not nullable in the declared schema;
- True allows a null value in the declared schema.

A declared non-nullable field does not remove the need to validate the input. Reader behavior for malformed records must be checked.

## 66. Read using the schema

~~~python
orders_typed = spark.read.csv(
    "data/orders.csv",
    header=True,
    schema=orders_schema
)

orders_typed.printSchema()
orders_typed.show()
~~~

Expected schema includes:

~~~text
 |-- order_id: integer (nullable = true)
 |-- customer_id: string (nullable = true)
 |-- city: string (nullable = true)
 |-- amount: decimal(12,2) (nullable = true)
~~~

CSV readers may represent parsed fields as nullable even when the supplied StructField was declared non-nullable. Verify actual behavior for your Spark version and reader options. Enforce business-required fields with explicit data-quality checks.

## 67. Validate null values

~~~python
from pyspark.sql.functions import col

orders_typed.select(
    [
        col(column).isNull().cast("int").sum().alias(column + "_nulls")
        for column in orders_typed.columns
    ]
).show()
~~~

Expected output for the supplied training CSV: zero nulls for all four columns.

Explanation:
- col(column) refers to a DataFrame column;
- isNull() identifies missing values;
- cast("int") converts true/false to a numeric indicator;
- sum() counts the missing indicators;
- alias() names the result column;
- select() creates the summary DataFrame;
- show() executes and displays it.

If the cell is unfamiliar, read it one expression at a time. You will learn transformations and functions in more detail in Day 56.

## 68. Why schema validation is not just printSchema

printSchema shows the current structure. It does not prove:
- every required field is present in the source contract;
- every row has a valid value;
- order IDs are unique;
- amounts are positive;
- city values belong to an approved list.

A production pipeline needs both structural checks and business-data checks.

---

# Part 16 — Understand Spark execution through actions

## 69. A DataFrame is not simply a Python list

When you write:

~~~python
paid_orders = orders.filter(orders.status == "PAID")
~~~

Spark builds a DataFrame transformation. It does not immediately return a Python list containing every matching row.

The variable represents a distributed dataset and its planned operations.

## 70. Trigger execution with an action

~~~python
paid_orders.show()
~~~

show() asks Spark to execute the necessary plan and display rows.

Other common actions include:
- count();
- collect();
- take();
- first();
- write operations.

## 71. Inspect the execution plan

~~~python
orders.explain()
~~~

This displays a query plan for the DataFrame.

The plan can show how Spark intends to read and process the data. It is not a timing benchmark by itself.

You will learn how to read plans and optimize transformations in later Spark lessons.

## 72. Understand the Spark UI

When a local Spark application is running, Spark commonly exposes a web UI on a local port such as 4040, if that port is available and UI is enabled.

The UI can show:
- jobs;
- stages;
- tasks;
- storage;
- environment;
- executors.

The exact page and available information depend on Spark version and deployment mode.

If the port is already in use, Spark may select another available port and print it in the application logs. Use the address reported by your running application.

Do not expose a production Spark UI publicly. It can reveal application and execution details.

---

# Part 17 — Understand local mode and cluster deployment more deeply

## 73. Local mode

The setting:

~~~python
.master("local[*]")
~~~

runs Spark on the current machine and uses available local cores.

It is appropriate for:
- learning DataFrame syntax;
- testing a small transformation;
- validating a sample;
- debugging file paths;
- developing before cluster deployment.

## 74. Cluster deployment

In a cluster deployment:
- a driver coordinates the application;
- a cluster manager allocates resources;
- executors run tasks;
- tasks process data partitions;
- results and status are coordinated.

The driver may run on the submitting machine or in a cluster-managed location, depending on deploy mode and platform.

Do not assume the driver always runs on your laptop.

## 75. Why local and cluster results can differ operationally

The business result should remain correct, but operational behavior can differ because:
- local files may not exist on cluster workers;
- cloud credentials may differ;
- network access may differ;
- available memory and cores differ;
- file permissions differ;
- dependency packages may differ;
- partition counts and scheduling differ.

A script that works locally still needs deployment testing.

---

# Part 18 — Common errors and how to investigate them

| Error or symptom | Possible reason | First check |
|---|---|---|
| pyspark not found | Package installed in another Python environment | Check interpreter and pip location |
| Java gateway exited | Java compatibility or environment issue | Verify supported Java and JAVA_HOME |
| File not found | Wrong working directory or path | Print os.getcwd() and inspect file |
| Cannot infer schema | Empty/malformed input or missing data | Inspect file and define schema |
| Amount inferred as integer | Sample has only whole numbers | Use DecimalType |
| Unexpected nulls | Parsing or source-quality issue | Inspect malformed records and schema |
| Spark UI not opening | UI disabled, app stopped or port changed | Read application startup output |
| Slow first action | Session startup and file scan | Separate startup from repeated query timing |
| Driver out of memory | Too much data brought to driver | Avoid collect() on large data |
| Permission denied on cloud file | Workload identity lacks storage access | Check approved cloud identity and resource policy |

Do not solve every problem by reinstalling everything. Identify the layer that failed.

---

# Part 19 — Practice exercises with worked answers

## Exercise 9 — SparkSession

Why does a PySpark program create a SparkSession?

**Worked answer:** It provides the main entry point for DataFrame and SQL operations and manages the Spark application session.

## Exercise 10 — Local mode

The code uses local[*]. Can the student claim that three cloud worker nodes processed the file?

**Worked answer:** No. local[*] requests local execution on the current machine. It does not create cloud worker nodes.

## Exercise 11 — Schema inference

A sample file has amount values 100 and 200. A later file contains 200.75. Why can inference be risky?

**Worked answer:** The first file may infer an integer type while the later data requires a decimal. Define and validate the intended type.

## Exercise 12 — Count

Why can count() take time even though it prints only one number?

**Worked answer:** Spark may need to scan the input and execute the plan to count all rows.

## Exercise 13 — collect

A DataFrame contains 50 million rows. Should you call collect() to inspect it?

**Worked answer:** No. It can transfer all rows to driver memory and cause an out-of-memory failure. Use show with a small limit or distributed summaries.

## Exercise 14 — Missing file

Spark cannot find data/orders.csv. What should you inspect first?

**Worked answer:** Current working directory, file name, extension and relative path.

## Exercise 15 — DataFrame

Is a Spark DataFrame the same as a pandas DataFrame?

**Worked answer:** No. Both organize data into columns, but Spark DataFrames use Spark's execution engine and are designed for distributed processing.

## Exercise 16 — Spark UI

Why might localhost:4040 not open?

**Worked answer:** The application may have stopped, the UI may be disabled, or the port may have been occupied and Spark selected another port. Check startup output.

## Exercise 17 — Local deployment

A Spark script works on a laptop but fails on a cluster. Give two possible reasons.

**Worked answer:** Cluster workers may not have the same file path or packages; the cluster may use a different identity or network route. Compare environment, dependencies, storage and permissions.

---

# Part 20 — Day 55 practical completion checklist

### Environment
- [ ] Check Python and pip versions.
- [ ] Check Java runtime compatibility.
- [ ] Install PySpark in the active environment.
- [ ] Create the practice folder and CSV.
- [ ] Confirm the current working directory.

### Spark basics
- [ ] Import SparkSession.
- [ ] Create a local SparkSession.
- [ ] Inspect Spark version, application name and master.
- [ ] Explain driver, executor and cluster manager.
- [ ] Explain local mode versus cluster mode.

### DataFrame practice
- [ ] Read a CSV file.
- [ ] Display a limited number of rows.
- [ ] Inspect schema, columns and data types.
- [ ] Count rows and columns.
- [ ] Describe a numeric field.
- [ ] Define an explicit schema.
- [ ] Check null values.
- [ ] Explain why inferred schema may be unsafe.

### Execution understanding
- [ ] Explain transformations and actions at a high level.
- [ ] Run explain() and recognize it as a plan.
- [ ] Locate the Spark UI when available.
- [ ] Explain why collect() can exhaust driver memory.
- [ ] Explain why local success does not prove cluster readiness.

## Final self-assessment

Without looking at the notes, explain:

1. What is Apache Spark, and what does PySpark provide?
2. What is the role of the driver?
3. What is the role of an executor?
4. What does a cluster manager do?
5. What is a SparkSession?
6. What is a DataFrame schema?
7. Why can inferSchema create inconsistent types?
8. What is the difference between a transformation and an action?
9. Why can count() trigger processing?
10. Why is collect() risky for large datasets?
11. What does local[*] mean?
12. Why can a locally working job fail in a cluster?

If you can create the Spark session, read the training CSV, inspect its schema and explain the execution model, you are ready for Day 56.

**Next lesson — Day 56: Spark Transformations and Actions.** We will use select, filter, withColumn, when, groupBy, agg, orderBy and distinct, inspect the results after each step, and understand how actions trigger execution.
