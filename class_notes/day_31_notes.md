# Day 31: File-Based Data Ingestion

**Phase:** 4 — Data Processing and Pipelines  
**Approved syllabus topic:** File-Based Data Ingestion  
**Syllabus coverage:** File ingestion, directory scanning, multiple files, file naming, ingestion validation, file processing automation  
**Mini-project:** Multi-File Ingestion System  
**Audience:** Freshers and complete beginners  
**Suggested class time:** 2–3 hours  
**Primary language:** Python  
**Formats used today:** CSV and JSON  
**Prerequisites:** Python file operations, lists, dictionaries, loops, exception handling and data formats from Phase 2

---

# 1. What will you learn today?

In Day 30, you learned how a pipeline extracts new or changed records from a database. But companies do not store all their data inside databases. Many business systems send data as files.

For example:
- A bank sends daily transaction files.
- An e-commerce company receives order files from its mobile application.
- A logistics partner sends delivery updates every hour.
- A hospital receives appointment files from its booking system.
- A finance team uploads monthly expense files.
- A marketing platform exports campaign data as CSV or JSON.

A Data Engineer must collect these files, identify which ones are ready, read them, validate their contents, and move them through a controlled process.

A file arriving in a folder does not automatically mean that it is safe to load.

Today's objective is to build a small but realistic file-ingestion system.

## 1.1 Learning outcomes

By the end of this lesson, you should be able to:

1. Explain file-based data ingestion.
2. Distinguish a source folder, landing folder, archive folder and rejected folder.
3. Explain why ingestion should not begin by blindly reading every file.
4. Create and inspect folders using Python's pathlib module.
5. List files in a directory.
6. Filter files by extension and naming convention.
7. Process multiple files in a predictable order.
8. Read CSV files with Python's csv module.
9. Read JSON files with Python's json module.
10. Validate file names, headers, required fields and values.
11. Distinguish a file-level error from a row-level data error.
12. Handle empty, malformed and unexpected files.
13. Avoid processing temporary or incomplete files.
14. Archive successful files and isolate rejected files.
15. Keep an ingestion manifest with processing status.
16. Understand duplicate-file handling and idempotency.
17. Build a multi-file ingestion project.
18. Explain the design in a junior Data Engineer interview.

## 1.2 Today's business scenario

Imagine that an online retail company receives order files from three sales channels:

- Website
- Mobile application
- Partner marketplace

Every channel sends CSV files to a shared folder.

Example files:

```text
incoming/
|
|-- orders_website_20261110.csv
|-- orders_mobile_20261110.csv
|-- orders_partner_20261110.csv
|-- readme.txt
|-- orders_website_20261111.tmp
```

The pipeline should process valid order CSV files. It should not accidentally read a text instruction file or a temporary file that is still being uploaded.

The flow is:

```text
Partner systems
      |
      v
Incoming folder
      |
      v
Discover eligible files
      |
      v
Validate file name and structure
      |
      v
Read CSV rows
      |
      v
Validate each order
      |
      +----------------------+
      |                      |
      v                      v
Valid records           Invalid records
      |                      |
      v                      v
Accepted output          Rejection report
      |
      v
Archive processed file
      |
      v
Record ingestion status
```

This is a small batch file-ingestion workflow.

---

# 2. What is file-based data ingestion?

**File-based data ingestion** is the process of collecting data from files and bringing that data into a system where it can be processed, stored or analyzed.

The source file may be:
- CSV
- JSON
- XML
- Excel
- Parquet
- Text or log file

The ingestion process may include:
1. Discovering a file.
2. Checking that it is the expected file.
3. Checking that the file is complete.
4. Reading the data.
5. Validating the structure and values.
6. Recording what happened.
7. Moving or storing the file safely.
8. Making the data available to the next pipeline step.

## 2.1 Ingestion is more than reading a file

Consider this code:

```python
file = open("orders.csv", "r")
print(file.read())
file.close()
```

This code opens and reads a file. It does not yet provide a complete ingestion process.

It does not answer:
- Is this the correct file?
- Is the file empty?
- Does it have the expected columns?
- Are the order IDs valid?
- Are amounts numeric?
- Has this file already been processed?
- What should happen if reading fails?
- Where should the file go after processing?

A production-oriented ingestion process needs these controls.

## 2.2 File ingestion versus file processing

| Term | Meaning |
|---|---|
| File discovery | Find files that may be ready for processing |
| File ingestion | Bring file data into the platform |
| File validation | Check whether the file and its records meet requirements |
| File transformation | Change the data into a required structure |
| File archival | Preserve a successfully processed source file |
| Rejection handling | Isolate a file or row that cannot be accepted |
| Manifest | Record what happened to each file |

These activities are connected, but they are not the same operation.

---

# 3. Understand the folders in an ingestion system

A beginner often stores every file in one folder. That is fine for a tiny exercise, but it becomes confusing when files arrive every day.

A better structure separates files according to their processing state.

## 3.1 Incoming folder

This is where source files arrive.

```text
data/
    incoming/
```

The pipeline discovers candidate files here.

Do not let unrelated files, scripts, logs and output files accumulate in the same directory.

## 3.2 Processing or working folder

A pipeline may copy or atomically move a file into a working folder before reading it.

```text
data/
    incoming/
    processing/
```

This can prevent two workers from picking up the same file at the same time, if the move/claim operation is designed safely.

For today's single-process classroom project, we will keep the design simpler. We will process files from incoming and move them after a result is recorded.

## 3.3 Archive folder

Successfully processed source files should usually be preserved.

```text
data/
    archive/
```

Why archive?
- Helps investigate a previous batch.
- Provides evidence of what the source sent.
- Supports controlled reprocessing.
- Keeps the incoming folder manageable.

An archive is not a substitute for a proper retention policy or backup.

## 3.4 Rejected folder

Files that fail file-level validation can be moved to a rejected area.

```text
data/
    rejected/
```

Examples:
- File has the wrong extension.
- CSV cannot be parsed.
- Required columns are missing.
- File is empty.
- File name does not follow the agreed convention.

A rejected file should not simply disappear. Record why it was rejected.

## 3.5 Output folder

The accepted records may be written to a prepared output file.

```text
data/
    output/
```

In a larger system, this output could be a database table, object-storage location or message queue. Today's project writes a consolidated CSV so that students can inspect the result easily.

## 3.6 Complete project folder structure

```text
day31_multi_file_ingestion/
|
|-- data/
|   |-- incoming/
|   |-- archive/
|   |-- rejected/
|   |-- output/
|
|-- reports/
|
|-- 01_create_sample_files.py
|-- 02_scan_directory.py
|-- 03_read_multiple_csv.py
|-- 04_validate_files.py
|-- 05_ingestion_pipeline.py
|-- README.md
```

We will create this structure step by step.

---

# 4. Prepare the development environment

Use the Python environment you configured in Phase 2.

## 4.1 Create the project folder

On Windows Command Prompt:

```bat
mkdir day31_multi_file_ingestion
cd day31_multi_file_ingestion
```

Create the main folders:

```bat
mkdir data
mkdir data\incoming
mkdir data\archive
mkdir data\rejected
mkdir data\output
mkdir reports
```

Your folder structure should now contain the directories shown in Section 3.6.

## 4.2 Do we need to install a new library?

For the first version, no.

We will use Python standard-library modules:

| Module | Purpose |
|---|---|
| pathlib | Work with file paths and directories |
| csv | Read and write CSV files |
| json | Read JSON files |
| shutil | Move or copy files |
| datetime | Work with processing timestamps |
| hashlib | Calculate a file fingerprint |
| os | Work with environment and operating-system details |

These modules are included with Python. You do not need to run pip install for them.

## 4.3 Check Python

Run:

```bat
python --version
```

Example output:

```text
Python 3.12.0
```

Your installed version may be different. The exact version is not important as long as it is a supported Python 3 version and your code runs.

---

# 5. Understand file paths before scanning folders

A path tells Python where a file or directory is located.

Example:

```text
day31_multi_file_ingestion/data/incoming/orders_website_20261110.csv
```

Break it down:

| Part | Meaning |
|---|---|
| day31_multi_file_ingestion | Project folder |
| data | Data folder |
| incoming | Source landing folder |
| orders_website_20261110.csv | File name |

## 5.1 Absolute path

An absolute path describes a location from the root of the file system.

Windows example:

```text
C:\Users\Student\day31_multi_file_ingestion\data\incoming
```

An absolute path can be tied to one computer or one user's folder.

## 5.2 Relative path

A relative path describes a location from the current working directory or from a known project location.

Example:

```text
data/incoming/orders_website_20261110.csv
```

Relative paths are useful for projects that should run on different computers.

## 5.3 Why use pathlib?

Older Python examples often use string paths and the os module. The pathlib module provides objects and methods designed for filesystem paths.

Example:

```python
from pathlib import Path

incoming_folder = Path("data/incoming")

print(incoming_folder)
print(incoming_folder.exists())
print(incoming_folder.is_dir())
```

Expected output when the folder exists:

```text
data/incoming
True
True
```

Explanation:

- from pathlib import Path imports the Path class.
- Path("data/incoming") creates a path object.
- exists() checks whether that path exists.
- is_dir() checks whether the path points to a directory.

A Path object is not the file itself. It is an object representing a filesystem location.

## 5.4 Create a folder from Python

```python
from pathlib import Path

incoming_folder = Path("data/incoming")

incoming_folder.mkdir(parents=True, exist_ok=True)

print("Incoming folder is ready")
```

Expected output:

```text
Incoming folder is ready
```

Understand the arguments:

- mkdir() creates a directory.
- parents=True creates missing parent directories too.
- exist_ok=True avoids an error if the directory already exists.

This makes the setup repeatable.

## 5.5 Check whether a path is a file or directory

```python
from pathlib import Path

folder_path = Path("data/incoming")
file_path = Path("data/incoming/orders.csv")

print("Folder exists:", folder_path.exists())
print("Folder is directory:", folder_path.is_dir())
print("File is a file:", file_path.is_file())
```

The last result is False if orders.csv has not been created.

Do not assume that a path exists just because you typed its name.

---

# 6. Create sample source files

We need sample files before learning how to scan them.

The company receives three order files. We will create them using Python so every student can reproduce the same exercise.

## 6.1 Understand the sample CSV structure

CSV means Comma-Separated Values.

Example:

```csv
order_id,customer_name,amount,order_status
1001,Asha,1500.00,PAID
1002,Ravi,2800.00,SHIPPED
```

The first line contains column names. The following lines contain records.

## 6.2 Create the first CSV file manually

Create a file named:

```text
data/incoming/orders_website_20261110.csv
```

Enter:

```csv
order_id,customer_name,amount,order_status
1001,Asha,1500.00,PAID
1002,Ravi,2800.00,SHIPPED
1003,Meena,900.00,PENDING
```

Save the file.

## 6.3 Create the second CSV file

File name:

```text
data/incoming/orders_mobile_20261110.csv
```

Data:

```csv
order_id,customer_name,amount,order_status
1004,Kiran,4500.00,PAID
1005,Divya,1200.00,PAID
1006,Arjun,3200.00,SHIPPED
```

## 6.4 Create the third CSV file

File name:

```text
data/incoming/orders_partner_20261110.csv
```

Data:

```csv
order_id,customer_name,amount,order_status
1007,Neha,700.00,PENDING
1008,Imran,2100.00,PAID
1009,John,1800.00,SHIPPED
```

## 6.5 What should the folder contain?

```text
data/
    incoming/
        orders_website_20261110.csv
        orders_mobile_20261110.csv
        orders_partner_20261110.csv
```

We have three files and three rows in each file. The total expected order count is nine.

This expected count is important. Later, we will compare the number of records we read with the number we expected.

## 6.6 Create the sample files automatically

Instead of creating them manually, use this script. This is useful when a student wants to reset the practice data.

Create 01_create_sample_files.py:

```python
from pathlib import Path
import csv

incoming_folder = Path("data/incoming")
incoming_folder.mkdir(parents=True, exist_ok=True)

website_orders = [
    [1001, "Asha", 1500.00, "PAID"],
    [1002, "Ravi", 2800.00, "SHIPPED"],
    [1003, "Meena", 900.00, "PENDING"]
]

mobile_orders = [
    [1004, "Kiran", 4500.00, "PAID"],
    [1005, "Divya", 1200.00, "PAID"],
    [1006, "Arjun", 3200.00, "SHIPPED"]
]

partner_orders = [
    [1007, "Neha", 700.00, "PENDING"],
    [1008, "Imran", 2100.00, "PAID"],
    [1009, "John", 1800.00, "SHIPPED"]
]

headers = ["order_id", "customer_name", "amount", "order_status"]

with open(incoming_folder / "orders_website_20261110.csv", "w", newline="", encoding="utf-8") as file:
    writer = csv.writer(file)
    writer.writerow(headers)
    writer.writerows(website_orders)

with open(incoming_folder / "orders_mobile_20261110.csv", "w", newline="", encoding="utf-8") as file:
    writer = csv.writer(file)
    writer.writerow(headers)
    writer.writerows(mobile_orders)

with open(incoming_folder / "orders_partner_20261110.csv", "w", newline="", encoding="utf-8") as file:
    writer = csv.writer(file)
    writer.writerow(headers)
    writer.writerows(partner_orders)

print("Three sample order files created")
```

Run:

```bat
python 01_create_sample_files.py
```

Expected output:

```text
Three sample order files created
```

### Understand the important code

**Path("data/incoming")**

Creates a Path object for the incoming directory.

**mkdir(parents=True, exist_ok=True)**

Ensures the directory exists before writing files.

**website_orders, mobile_orders, partner_orders**

These are Python lists. Each inner list represents one order row.

**headers**

Contains the CSV column names.

**open(path, "w", newline="", encoding="utf-8")**

- "w" opens the file for writing and replaces its existing contents.
- newline="" is recommended when using Python's csv module.
- encoding="utf-8" provides consistent text encoding.

**csv.writer(file)**

Creates a CSV writer associated with the open file.

**writerow(headers)**

Writes one row: the column names.

**writerows(website_orders)**

Writes all rows from the list.

**with**

The with statement manages the file handle and closes it when the block finishes, even if an exception occurs.

### Important warning

This is a sample-data creation script. It intentionally overwrites the three sample files each time it runs. Do not copy that overwrite behavior into a production ingestion process.

---

# 7. Scan a directory

A Data Engineer cannot manually type every incoming file name into the pipeline. The pipeline needs to discover files automatically.

## 7.1 List all entries in a directory

Create 02_scan_directory.py:

```python
from pathlib import Path

incoming_folder = Path("data/incoming")

for item in incoming_folder.iterdir():
    print(item.name)
```

Expected output (the order can vary by filesystem):

```text
orders_website_20261110.csv
orders_mobile_20261110.csv
orders_partner_20261110.csv
```

Explanation:

- iterdir() returns the entries directly inside the folder.
- item represents one entry.
- item.name returns only the entry's name, not its full path.

The order of entries returned by iterdir() is not guaranteed. If processing order matters, sort the results.

## 7.2 Print full paths

```python
from pathlib import Path

incoming_folder = Path("data/incoming")

for item in incoming_folder.iterdir():
    print(item)
```

The output includes the relative path, such as:

```text
data/incoming/orders_website_20261110.csv
```

The difference:
- item.name -> file name only.
- item -> path object for the entry.

## 7.3 List only files

A directory may contain subdirectories. We should not try to read a folder as if it were a CSV file.

```python
from pathlib import Path

incoming_folder = Path("data/incoming")

for item in incoming_folder.iterdir():
    if item.is_file():
        print(item.name)
```

The is_file() check filters out directories.

## 7.4 List only CSV files

```python
from pathlib import Path

incoming_folder = Path("data/incoming")

for item in incoming_folder.iterdir():
    if item.is_file() and item.suffix.lower() == ".csv":
        print(item.name)
```

Expected output: the three CSV file names.

Explanation:
- suffix returns the file extension, including the dot.
- lower() makes the extension comparison case-insensitive.
- ".CSV" and ".csv" are treated the same.

## 7.5 Use glob to find matching files

```python
from pathlib import Path

incoming_folder = Path("data/incoming")

csv_files = incoming_folder.glob("*.csv")

for file_path in sorted(csv_files):
    print(file_path.name)
```

Expected output:

```text
orders_mobile_20261110.csv
orders_partner_20261110.csv
orders_website_20261110.csv
```

glob("*.csv") means:
- * matches any sequence of characters.
- .csv requires the file name to end in .csv.

sorted() gives a predictable alphabetical order.

## 7.6 What is the difference between iterdir and glob?

| Method | Meaning | Example |
|---|---|---|
| iterdir() | Return direct entries in a directory | Every file and subfolder |
| glob("*.csv") | Return entries matching a pattern | CSV files |
| rglob("*.csv") | Search matching files recursively | CSV files in nested folders |

Use the simplest method that matches the requirement.

## 7.7 Scan nested directories

Suppose the folder structure is:

```text
incoming/
    website/
        website_orders.csv
    mobile/
        mobile_orders.csv
```

Use:

```python
from pathlib import Path

incoming_folder = Path("data/incoming")

for file_path in incoming_folder.rglob("*.csv"):
    print(file_path)
```

rglob searches subdirectories too.

Be careful: recursive scanning can pick up files from old or unrelated folders if your directory design is not controlled.

---

# 8. File naming conventions

A file name can carry useful information.

Example:

```text
orders_website_20261110.csv
```

Possible parts:

| Part | Meaning |
|---|---|
| orders | Dataset |
| website | Source system |
| 20261110 | Business or delivery date |
| csv | File format |

A naming convention helps the pipeline identify:
- Dataset name.
- Source system.
- Date or batch.
- File format.
- Version or sequence, when needed.

## 8.1 Good file names

Examples:

```text
orders_website_20261110.csv
orders_mobile_20261110.csv
customers_website_20261110.csv
payments_partner_20261110.csv
```

They are descriptive and consistent.

## 8.2 Poor file names

Examples:

```text
file1.csv
newfile.csv
final.csv
final_latest_revised.csv
data.csv
```

These names do not clearly identify the dataset, source or delivery period.

## 8.3 Check a file name

```python
from pathlib import Path

file_path = Path("data/incoming/orders_website_20261110.csv")

print(file_path.name)
print(file_path.stem)
print(file_path.suffix)
```

Expected output:

```text
orders_website_20261110.csv
orders_website_20261110
.csv
```

- name returns the complete file name.
- stem returns the name without the extension.
- suffix returns the extension.

## 8.4 Validate a naming pattern

For this lesson, our required pattern is:

```text
orders_<source>_<YYYYMMDD>.csv
```

We will use Python's re module to validate the pattern. A regular expression is a text pattern. Do not worry about memorizing every symbol immediately.

```python
import re

file_name = "orders_website_20261110.csv"

pattern = r"^orders_[a-z]+_\d{8}\.csv$"

if re.match(pattern, file_name):
    print("Valid file name")
else:
    print("Invalid file name")
```

Expected output:

```text
Valid file name
```

Understand the pattern:

| Part | Meaning |
|---|---|
| ^ | Start of the file name |
| orders_ | Exact text required at the start |
| [a-z]+ | One or more lowercase letters for source name |
| _ | Required underscore |
| \d{8} | Exactly eight digits for the date |
| \.csv | Literal .csv extension |
| $ | End of the file name |

This is a teaching pattern. A production naming contract may allow hyphens, uppercase source names, multiple source tokens or a different date format.

## 8.5 Validate the date in the name

Eight digits do not guarantee a valid calendar date. For example, 20261345 has eight digits but is not a valid date.

```python
from datetime import datetime

date_text = "20261110"

file_date = datetime.strptime(date_text, "%Y%m%d").date()

print(file_date)
```

Expected output:

```text
2026-11-10
```

If the date is invalid, strptime raises ValueError. A production pipeline should catch that error and record a useful rejection reason.

---

# 9. Read a single CSV file

Before reading many files, understand one file clearly.

## 9.1 What is a CSV row?

Example:

```csv
order_id,customer_name,amount,order_status
1001,Asha,1500.00,PAID
1002,Ravi,2800.00,SHIPPED
```

The first row is the header. Each following row contains values in the corresponding columns.

## 9.2 Read CSV using csv.reader

```python
import csv

with open(
    "data/incoming/orders_website_20261110.csv",
    "r",
    newline="",
    encoding="utf-8"
) as file:

    reader = csv.reader(file)

    for row in reader:
        print(row)
```

Expected output:

```text
['order_id', 'customer_name', 'amount', 'order_status']
['1001', 'Asha', '1500.00', 'PAID']
['1002', 'Ravi', '2800.00', 'SHIPPED']
['1003', 'Meena', '900.00', 'PENDING']
```

Important observations:
- Each row is returned as a list.
- CSV values are read as strings.
- The first row is the header.
- The reader does not automatically convert "1500.00" into a number.

## 9.3 Skip the header

```python
import csv

with open(
    "data/incoming/orders_website_20261110.csv",
    "r",
    newline="",
    encoding="utf-8"
) as file:

    reader = csv.reader(file)

    headers = next(reader)

    print("Headers:", headers)

    for row in reader:
        print("Data row:", row)
```

Expected output:

```text
Headers: ['order_id', 'customer_name', 'amount', 'order_status']
Data row: ['1001', 'Asha', '1500.00', 'PAID']
Data row: ['1002', 'Ravi', '2800.00', 'SHIPPED']
Data row: ['1003', 'Meena', '900.00', 'PENDING']
```

next(reader) reads the next item from the reader. Here it reads the header, so the loop begins with the first data row.

## 9.4 Read CSV using DictReader

DictReader maps each row to a dictionary using the header names as keys.

```python
import csv

with open(
    "data/incoming/orders_website_20261110.csv",
    "r",
    newline="",
    encoding="utf-8"
) as file:

    reader = csv.DictReader(file)

    for row in reader:
        print(row)
```

Expected output:

```text
{'order_id': '1001', 'customer_name': 'Asha', 'amount': '1500.00', 'order_status': 'PAID'}
{'order_id': '1002', 'customer_name': 'Ravi', 'amount': '2800.00', 'order_status': 'SHIPPED'}
{'order_id': '1003', 'customer_name': 'Meena', 'amount': '900.00', 'order_status': 'PENDING'}
```

A dictionary is easier to understand when records have named columns.

For example:

```python
print(row["customer_name"])
print(row["amount"])
```

For each current row, this prints the customer name and amount.

## 9.5 Convert values to the correct types

CSV readers return text. If we need to calculate amounts, convert the text to a numeric type.

```python
amount_text = "1500.00"

amount = float(amount_text)

print(amount)
print(amount * 2)
```

Expected output:

```text
1500.0
3000.0
```

For financial applications, decimal.Decimal is generally preferable to binary float because decimal currency values should not inherit binary floating-point representation surprises. We will introduce that distinction in the project extension.

---

# 10. Read multiple CSV files

Now we move from one file to all eligible files in the incoming directory.

## 10.1 Discover CSV files

```python
from pathlib import Path

incoming_folder = Path("data/incoming")

csv_files = sorted(incoming_folder.glob("*.csv"))

print("Files found:", len(csv_files))

for file_path in csv_files:
    print(file_path.name)
```

Expected output:

```text
Files found: 3
orders_mobile_20261110.csv
orders_partner_20261110.csv
orders_website_20261110.csv
```

## 10.2 Read every file

```python
import csv
from pathlib import Path

incoming_folder = Path("data/incoming")

csv_files = sorted(incoming_folder.glob("*.csv"))

for file_path in csv_files:
    print("Reading file:", file_path.name)

    with open(file_path, "r", newline="", encoding="utf-8") as file:
        reader = csv.DictReader(file)

        for row in reader:
            print(row)
```

The outer loop handles files. The inner loop handles records inside the current file.

```text
Outer loop: file 1
    Inner loop: row 1
    Inner loop: row 2
    Inner loop: row 3

Outer loop: file 2
    Inner loop: row 1
    Inner loop: row 2
    Inner loop: row 3

Outer loop: file 3
    Inner loop: row 1
    Inner loop: row 2
    Inner loop: row 3
```

This nested-loop pattern is common in file ingestion.

## 10.3 Count rows in every file

```python
import csv
from pathlib import Path

incoming_folder = Path("data/incoming")

total_rows = 0

for file_path in sorted(incoming_folder.glob("*.csv")):
    file_rows = 0

    with open(file_path, "r", newline="", encoding="utf-8") as file:
        reader = csv.DictReader(file)

        for row in reader:
            file_rows = file_rows + 1

    print(file_path.name, "->", file_rows, "rows")

    total_rows = total_rows + file_rows

print("Total rows:", total_rows)
```

Expected output:

```text
orders_mobile_20261110.csv -> 3 rows
orders_partner_20261110.csv -> 3 rows
orders_website_20261110.csv -> 3 rows
Total rows: 9
```

Understand the counters:

- total_rows starts at zero before any file is processed.
- file_rows resets to zero for each file.
- Each row increases file_rows by one.
- After a file is read, its count is added to total_rows.

If total_rows were placed inside the outer loop, it would reset for every file. That is a common beginner mistake.

---

# 11. File-level validation

Before processing data, validate the file itself.

File-level validation checks the whole file or its structure. Examples:
- Correct extension.
- Correct naming convention.
- File exists.
- File is not empty.
- File can be opened.
- Expected header columns are present.
- File is not still being written.

## 11.1 Check that a file exists

```python
from pathlib import Path

file_path = Path("data/incoming/orders_website_20261110.csv")

if file_path.exists():
    print("File exists")
else:
    print("File does not exist")
```

Expected output if the sample file exists:

```text
File exists
```

## 11.2 Check whether a file is empty

```python
from pathlib import Path

file_path = Path("data/incoming/orders_website_20261110.csv")

if file_path.stat().st_size == 0:
    print("File is empty")
else:
    print("File contains data")
```

Expected output:

```text
File contains data
```

stat().st_size returns file size in bytes. A non-zero file can still contain only a header or invalid content, so this check alone is not enough.

## 11.3 Check the expected headers

Our required headers are:

```python
expected_headers = [
    "order_id",
    "customer_name",
    "amount",
    "order_status"
]
```

Read the actual headers:

```python
import csv
from pathlib import Path

file_path = Path("data/incoming/orders_website_20261110.csv")

with open(file_path, "r", newline="", encoding="utf-8") as file:
    reader = csv.reader(file)
    actual_headers = next(reader)

print("Actual headers:", actual_headers)
```

Expected output:

```text
Actual headers: ['order_id', 'customer_name', 'amount', 'order_status']
```

Compare:

```python
if actual_headers == expected_headers:
    print("Header structure is correct")
else:
    print("Header structure is incorrect")
```

Expected output:

```text
Header structure is correct
```

This exact comparison also checks column order. Some systems only require the same column names, regardless of order. In that case compare sets or explicitly map fields, but be careful: sets do not preserve column order and do not detect duplicate column names.

## 11.4 Check required columns without enforcing order

```python
required_columns = {
    "order_id",
    "customer_name",
    "amount",
    "order_status"
}

actual_columns = set(actual_headers)

missing_columns = required_columns - actual_columns

if len(missing_columns) == 0:
    print("All required columns are present")
else:
    print("Missing columns:", missing_columns)
```

The set difference finds required names that do not exist in the actual header.

If extra columns are allowed, this approach is useful. If exact schema matching is required, check both missing and unexpected columns.

## 11.5 Detect duplicate column names

A CSV header can accidentally contain the same column twice.

```python
headers = ["order_id", "customer_name", "amount", "amount"]

if len(headers) != len(set(headers)):
    print("Duplicate column names found")
else:
    print("No duplicate column names")
```

Expected output:

```text
Duplicate column names found
```

Why check this? Duplicate names can make dictionary-based parsing ambiguous or cause one value to overwrite another.

---

# 12. Row-level validation

A file can have a correct name and header but still contain invalid records.

Example:

```csv
order_id,customer_name,amount,order_status
1001,Asha,1500.00,PAID
1002,,2800.00,SHIPPED
1003,Meena,not_available,PENDING
1004,Kiran,-500.00,PAID
```

Problems:
- Row 1002 has no customer name.
- Row 1003 has a non-numeric amount.
- Row 1004 has a negative amount, which violates this practice business rule.

A pipeline should identify these records rather than silently accepting them.

## 12.1 Define the business rules

For this classroom order feed:

| Field | Rule |
|---|---|
| order_id | Required positive integer |
| customer_name | Required non-empty text |
| amount | Required numeric value greater than or equal to zero |
| order_status | Must be PAID, SHIPPED or PENDING |

These are example business rules. A real company must agree the rules with the business owner.

## 12.2 Validate one record manually

```python
row = {
    "order_id": "1001",
    "customer_name": "Asha",
    "amount": "1500.00",
    "order_status": "PAID"
}

if row["order_id"].isdigit():
    print("Order ID format is valid")

if row["customer_name"].strip() != "":
    print("Customer name is present")

if row["order_status"] in ["PAID", "SHIPPED", "PENDING"]:
    print("Order status is allowed")
```

Expected output:

```text
Order ID format is valid
Customer name is present
Order status is allowed
```

We have not yet checked amount. We will do that next.

## 12.3 Validate amount safely

```python
amount_text = "1500.00"

try:
    amount = float(amount_text)

    if amount >= 0:
        print("Amount is valid")
    else:
        print("Amount cannot be negative")

except ValueError:
    print("Amount must be numeric")
```

Expected output:

```text
Amount is valid
```

The try block contains the conversion that may fail. If the text is not numeric, float() raises ValueError and the except block handles it.

## 12.4 Validate a bad amount

Change the value:

```python
amount_text = "not_available"
```

The output becomes:

```text
Amount must be numeric
```

This is an example of handling invalid source data without crashing the entire teaching exercise.

## 12.5 Build a row validation example

```python
row = {
    "order_id": "1001",
    "customer_name": "Asha",
    "amount": "1500.00",
    "order_status": "PAID"
}

errors = []

if not row["order_id"].isdigit():
    errors.append("order_id must contain digits")

if row["customer_name"].strip() == "":
    errors.append("customer_name is required")

try:
    amount = float(row["amount"])

    if amount < 0:
        errors.append("amount cannot be negative")

except ValueError:
    errors.append("amount must be numeric")

if row["order_status"] not in ["PAID", "SHIPPED", "PENDING"]:
    errors.append("order_status is not allowed")

if len(errors) == 0:
    print("Record is valid")
else:
    print("Record is invalid")
    for error in errors:
        print("-", error)
```

Expected output for the valid record:

```text
Record is valid
```

The errors list collects all detected issues. This is more useful than stopping after the first invalid field because a source owner can correct multiple issues in one response.

---

# 13. File-level errors versus row-level errors

These are different failure categories.

| Failure | Example | Possible handling |
|---|---|---|
| File-level | CSV cannot be parsed | Reject the entire file |
| File-level | Required header missing | Reject the entire file |
| File-level | Empty file | Reject or classify as empty delivery |
| Row-level | One amount is not numeric | Reject that row and keep valid rows, if contract allows |
| Row-level | Customer name is missing | Reject that row or apply an approved default |
| Row-level | Duplicate order ID | Apply agreed duplicate policy |

Do not automatically reject an entire file because one row is invalid unless the business contract says the file must be atomic.

Likewise, do not silently drop invalid rows. Record the reason and counts.

## 13.1 Example: file with valid and invalid rows

| order_id | customer_name | amount | order_status | Result |
|---:|---|---:|---|---|
| 2001 | Asha | 1500 | PAID | Valid |
| 2002 | Ravi | 2800 | SHIPPED | Valid |
| 2003 |  | 900 | PENDING | Invalid name |
| 2004 | Meena | ABC | PAID | Invalid amount |

Possible processing result:
- 2 accepted rows.
- 2 rejected rows.

The pipeline should report both counts.

---

# 14. Read JSON files

CSV is not the only file format used in ingestion.

JSON represents data using objects, arrays, strings, numbers, booleans and null values.

Example:

```json
{
  "source": "website",
  "orders": [
    {
      "order_id": 3001,
      "customer_name": "Asha",
      "amount": 1500.0,
      "order_status": "PAID"
    },
    {
      "order_id": 3002,
      "customer_name": "Ravi",
      "amount": 2800.0,
      "order_status": "SHIPPED"
    }
  ]
}
```

## 14.1 Create a JSON sample file

Save this as:

```text
data/incoming/orders_website_20261110.json
```

Content:

```json
{
  "source": "website",
  "orders": [
    {
      "order_id": 3001,
      "customer_name": "Asha",
      "amount": 1500.0,
      "order_status": "PAID"
    },
    {
      "order_id": 3002,
      "customer_name": "Ravi",
      "amount": 2800.0,
      "order_status": "SHIPPED"
    }
  ]
}
```

## 14.2 Read JSON using json.load

```python
import json
from pathlib import Path

file_path = Path("data/incoming/orders_website_20261110.json")

with open(file_path, "r", encoding="utf-8") as file:
    data = json.load(file)

print(data)
```

The result is a Python dictionary containing:
- source
- orders

## 14.3 Read the nested orders list

```python
print("Source:", data["source"])

for order in data["orders"]:
    print(order)
```

Expected output:

```text
Source: website
{'order_id': 3001, 'customer_name': 'Asha', 'amount': 1500.0, 'order_status': 'PAID'}
{'order_id': 3002, 'customer_name': 'Ravi', 'amount': 2800.0, 'order_status': 'SHIPPED'}
```

Notice that JSON numbers are read as Python numeric values, unlike CSV fields which are read as strings.

## 14.4 CSV and JSON comparison

| Feature | CSV | JSON |
|---|---|---|
| Structure | Rows and columns | Objects and arrays |
| Nested records | Not naturally supported | Supported |
| Human readability | High for simple tables | High for hierarchical data |
| Data types | Text values in typical CSV readers | Supports strings, numbers, booleans and null |
| Common use | Tabular exports | APIs and nested application data |

The right format depends on the source and the downstream requirement.

---

# 15. Handle malformed files

A source file may be incomplete or invalid.

Examples:
- Invalid CSV quoting.
- Invalid JSON syntax.
- Wrong text encoding.
- File is still being uploaded.
- File has a header but no records.
- File is truncated.

## 15.1 Catch a JSON parsing error

```python
import json

json_text = '{"order_id": 1001, "amount": }'

try:
    data = json.loads(json_text)
    print(data)

except json.JSONDecodeError as error:
    print("Invalid JSON file")
    print(error)
```

Expected output begins with:

```text
Invalid JSON file
```

The exact parser error text depends on the Python version and the position of the invalid character.

Do not hide the exception without recording the file and reason.

## 15.2 Catch a missing file

```python
from pathlib import Path

file_path = Path("data/incoming/missing_orders.csv")

try:
    with open(file_path, "r", encoding="utf-8") as file:
        print(file.read())

except FileNotFoundError:
    print("Source file was not found")
```

Expected output:

```text
Source file was not found
```

A missing file may mean the partner has not delivered it yet, the name is wrong or the path is incorrect.

## 15.3 Do not treat every exception as the same

Bad practice:

```python
try:
    # Read and process file
    pass
except Exception:
    print("Something went wrong")
```

This hides important details.

Better practice:
- Catch expected exceptions where they can be handled.
- Record the file name.
- Record the error category and message.
- Do not mark the file successful if processing failed.
- Allow unexpected programming errors to remain visible during development.

---

# 16. Avoid reading incomplete files

A file may appear in the incoming directory before the source has finished writing it.

For example, a partner may upload a 500 MB CSV. The file name appears immediately, but the upload continues for several minutes.

If the pipeline reads it too early, it may see incomplete data.

Common strategies include:

1. Source uploads to a temporary extension such as .tmp and renames to .csv only after completion.
2. Source writes to a temporary directory and atomically moves the completed file into incoming.
3. Source sends a separate completion marker.
4. Pipeline waits for a documented delivery-complete signal.
5. Pipeline checks file stability as a supplementary safeguard.

A file-size check alone does not prove completeness. The source should provide a clear completion contract.

## 16.1 Ignore temporary files

```python
from pathlib import Path

incoming_folder = Path("data/incoming")

for file_path in sorted(incoming_folder.iterdir()):
    if file_path.is_file() and file_path.suffix.lower() == ".csv":
        print("Ready candidate:", file_path.name)
```

A file named orders.csv.tmp has suffix .tmp, so it is not selected.

However, a source that writes directly to a .csv file while uploading can still be picked up too early. File naming conventions should prevent that.

---

# 17. Archive successfully processed files

Once a file has been read and accepted according to the ingestion rules, the source file can be moved to an archive directory.

## 17.1 Move one file

```python
from pathlib import Path
import shutil

source_file = Path("data/incoming/orders_website_20261110.csv")
archive_folder = Path("data/archive")

archive_folder.mkdir(parents=True, exist_ok=True)

destination_file = archive_folder / source_file.name

shutil.move(str(source_file), str(destination_file))

print("File archived:", destination_file.name)
```

Expected output:

```text
File archived: orders_website_20261110.csv
```

Important:
- shutil.move moves a file or directory.
- str() converts the Path object to a string for the function.
- The destination path is built using the / operator on Path objects.

## 17.2 Avoid overwriting an archived file

A file with the same name may already exist in the archive.

Do not blindly overwrite evidence from a previous delivery.

A simple classroom check:

```python
if destination_file.exists():
    print("Archive file already exists")
else:
    shutil.move(str(source_file), str(destination_file))
    print("File archived")
```

Production systems should define how to handle repeated deliveries with the same name: reject, compare content, version the archive or treat as a duplicate according to the source contract.

## 17.3 Move rejected files separately

```python
from pathlib import Path
import shutil

source_file = Path("data/incoming/bad_orders.csv")
rejected_folder = Path("data/rejected")

rejected_folder.mkdir(parents=True, exist_ok=True)

destination_file = rejected_folder / source_file.name

if source_file.exists() and not destination_file.exists():
    shutil.move(str(source_file), str(destination_file))
    print("File moved to rejected folder")
```

A production system should also write a rejection report explaining why the file was rejected.

---

# 18. File identity and duplicate deliveries

A file name alone is not always a reliable identity.

A partner might:
- Send the same file twice.
- Send a corrected file with the same name.
- Send identical data under a different name.
- Re-send a file after not receiving an acknowledgement.

A robust ingestion system needs a duplicate policy.

## 18.1 Simple file fingerprint

A checksum or hash can help identify identical file contents.

```python
import hashlib
from pathlib import Path

file_path = Path("data/incoming/orders_website_20261110.csv")

file_hash = hashlib.sha256(file_path.read_bytes()).hexdigest()

print(file_hash)
```

The exact hash string depends on the file bytes.

SHA-256 produces a fixed-length fingerprint. If two files have identical bytes, they produce the same SHA-256 digest. A matching digest is strong evidence of identical content, though the hash is not a substitute for business-level duplicate checks.

For very large files, read and hash in chunks rather than loading the entire file into memory. We will revisit memory-efficient processing in later lessons.

## 18.2 File duplicate versus record duplicate

| Type | Example |
|---|---|
| Duplicate file | Same delivered file is sent twice |
| Duplicate record | Same order_id appears in two different files |
| Business duplicate | Two different order IDs represent the same real-world purchase due to a source-system error |

These need different rules.

A file hash detects identical file bytes. A primary/business key detects repeated records. Business duplicate detection may require more context.

---

# 19. Keep an ingestion manifest

A manifest is a record of files seen and their processing outcomes.

It may contain:

| Field | Purpose |
|---|---|
| file_name | Which file was processed |
| source_name | Which source delivered it |
| discovered_at | When the pipeline found it |
| processed_at | When processing completed |
| file_status | SUCCESS, REJECTED or FAILED |
| rows_read | Number of data rows read |
| rows_accepted | Number of valid rows |
| rows_rejected | Number of invalid rows |
| error_message | Reason for failure, if any |

A manifest makes a pipeline auditable.

## 19.1 Create a simple manifest CSV

```python
import csv
from pathlib import Path
from datetime import datetime

manifest_path = Path("reports/ingestion_manifest.csv")
manifest_path.parent.mkdir(parents=True, exist_ok=True)

with open(manifest_path, "w", newline="", encoding="utf-8") as file:
    writer = csv.writer(file)

    writer.writerow([
        "file_name",
        "processed_at",
        "file_status",
        "rows_read",
        "rows_accepted",
        "rows_rejected",
        "error_message"
    ])

    writer.writerow([
        "orders_website_20261110.csv",
        datetime.now().isoformat(timespec="seconds"),
        "SUCCESS",
        3,
        3,
        0,
        ""
    ])

print("Manifest created")
```

Expected output:

```text
Manifest created
```

The timestamp will vary based on the computer's clock and timezone.

This example recreates the manifest file. In the project, we will append a row for each processed file rather than overwriting the manifest every time.

## 19.2 Append manifest records

```python
import csv
from pathlib import Path
from datetime import datetime

manifest_path = Path("reports/ingestion_manifest.csv")
manifest_path.parent.mkdir(parents=True, exist_ok=True)

file_exists = manifest_path.exists()

with open(manifest_path, "a", newline="", encoding="utf-8") as file:
    writer = csv.writer(file)

    if not file_exists:
        writer.writerow([
            "file_name",
            "processed_at",
            "file_status",
            "rows_read",
            "rows_accepted",
            "rows_rejected",
            "error_message"
        ])

    writer.writerow([
        "orders_mobile_20261110.csv",
        datetime.now().isoformat(timespec="seconds"),
        "SUCCESS",
        3,
        3,
        0,
        ""
    ])

print("Manifest updated")
```

The "a" mode appends to the existing file. If the manifest does not exist, Python creates it.

For concurrent pipelines, a plain CSV manifest is not enough. Use a transactional database table or another concurrency-safe metadata store.

---

# 20. Automate file processing

We now have the building blocks:
- Discover files.
- Read CSV.
- Validate headers.
- Validate rows.
- Write accepted data.
- Record file status.
- Archive processed files.

The next step is to combine them into a repeatable pipeline.

Do not begin with one huge script. Understand the flow first.

## 20.1 Pipeline stages

```text
1. Set folder paths
       |
       v
2. Discover CSV files
       |
       v
3. Validate file name and size
       |
       v
4. Validate CSV header
       |
       v
5. Read records
       |
       v
6. Validate each record
       |
       +----------------------+
       |                      |
       v                      v
7. Accepted rows         Rejected rows
       |                      |
       v                      v
8. Consolidated CSV      Rejection report
       |
       v
9. Archive successful file
       |
       v
10. Write manifest
```

## 20.2 Define the output schema

The consolidated output will contain:

```csv
order_id,customer_name,amount,order_status,source_file
```

We add source_file so we know which input file supplied each record.

This is called lineage information. It helps trace a record back to its source.

## 20.3 Prepare an invalid sample file

Create:

```text
data/incoming/orders_test_20261110.csv
```

Content:

```csv
order_id,customer_name,amount,order_status
1010,Asha,1200.00,PAID
1011,,700.00,PENDING
1012,Ravi,not_numeric,PAID
1013,Meena,-200.00,SHIPPED
1014,Kiran,500.00,UNKNOWN
```

Expected:
- One valid record.
- Four invalid records.

We use this file to demonstrate row-level rejection.

## 20.4 Complete beginner pipeline script

Create 05_ingestion_pipeline.py.

The script below is intentionally written in a direct, readable style. We are not hiding the main learning flow inside multiple custom functions.

```python
import csv
import shutil
from pathlib import Path
from datetime import datetime

incoming_folder = Path("data/incoming")
archive_folder = Path("data/archive")
rejected_folder = Path("data/rejected")
output_folder = Path("data/output")
reports_folder = Path("reports")

archive_folder.mkdir(parents=True, exist_ok=True)
rejected_folder.mkdir(parents=True, exist_ok=True)
output_folder.mkdir(parents=True, exist_ok=True)
reports_folder.mkdir(parents=True, exist_ok=True)

expected_headers = [
    "order_id",
    "customer_name",
    "amount",
    "order_status"
]

allowed_statuses = ["PAID", "SHIPPED", "PENDING"]

accepted_rows = []
rejected_rows = []
manifest_rows = []

csv_files = sorted(incoming_folder.glob("orders_*.csv"))

print("Files discovered:", len(csv_files))

for file_path in csv_files:

    print("\nProcessing:", file_path.name)

    file_rows_read = 0
    file_rows_accepted = 0
    file_rows_rejected = 0
    file_errors = []

    if file_path.stat().st_size == 0:
        file_errors.append("File is empty")

    if len(file_errors) == 0:

        try:
            with open(file_path, "r", newline="", encoding="utf-8-sig") as file:

                reader = csv.DictReader(file)

                actual_headers = reader.fieldnames

                if actual_headers != expected_headers:
                    file_errors.append("Header does not match expected structure")

                else:
                    for row in reader:

                        file_rows_read = file_rows_read + 1
                        row_errors = []

                        order_id_text = row["order_id"].strip()
                        customer_name = row["customer_name"].strip()
                        amount_text = row["amount"].strip()
                        order_status = row["order_status"].strip().upper()

                        if not order_id_text.isdigit():
                            row_errors.append("order_id must be a positive integer")

                        elif int(order_id_text) <= 0:
                            row_errors.append("order_id must be greater than zero")

                        if customer_name == "":
                            row_errors.append("customer_name is required")

                        try:
                            amount = float(amount_text)

                            if amount < 0:
                                row_errors.append("amount cannot be negative")

                        except ValueError:
                            amount = None
                            row_errors.append("amount must be numeric")

                        if order_status not in allowed_statuses:
                            row_errors.append("order_status is not allowed")

                        if len(row_errors) == 0:

                            accepted_rows.append({
                                "order_id": int(order_id_text),
                                "customer_name": customer_name,
                                "amount": amount,
                                "order_status": order_status,
                                "source_file": file_path.name
                            })

                            file_rows_accepted = file_rows_accepted + 1

                        else:

                            rejected_rows.append({
                                "source_file": file_path.name,
                                "row_number": file_rows_read + 1,
                                "order_id": order_id_text,
                                "errors": "; ".join(row_errors)
                            })

                            file_rows_rejected = file_rows_rejected + 1

        except UnicodeDecodeError:
            file_errors.append("File encoding is not valid UTF-8")

        except csv.Error as error:
            file_errors.append("CSV parsing error: " + str(error))

        except OSError as error:
            file_errors.append("File access error: " + str(error))

    if len(file_errors) == 0:

        file_status = "SUCCESS"

        destination_file = archive_folder / file_path.name

        if destination_file.exists():
            file_status = "FAILED"
            file_errors.append("Archive destination already exists")
        else:
            shutil.move(str(file_path), str(destination_file))

    else:

        file_status = "REJECTED"

        destination_file = rejected_folder / file_path.name

        if destination_file.exists():
            file_errors.append("Rejected destination already exists")
        else:
            shutil.move(str(file_path), str(destination_file))

    manifest_rows.append({
        "file_name": file_path.name,
        "processed_at": datetime.now().isoformat(timespec="seconds"),
        "file_status": file_status,
        "rows_read": file_rows_read,
        "rows_accepted": file_rows_accepted,
        "rows_rejected": file_rows_rejected,
        "error_message": "; ".join(file_errors)
    })

    print("Status:", file_status)
    print("Rows read:", file_rows_read)
    print("Rows accepted:", file_rows_accepted)
    print("Rows rejected:", file_rows_rejected)

output_file = output_folder / "accepted_orders.csv"

with open(output_file, "w", newline="", encoding="utf-8") as file:

    fieldnames = [
        "order_id",
        "customer_name",
        "amount",
        "order_status",
        "source_file"
    ]

    writer = csv.DictWriter(file, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerows(accepted_rows)

rejection_file = reports_folder / "rejected_rows.csv"

with open(rejection_file, "w", newline="", encoding="utf-8") as file:

    fieldnames = [
        "source_file",
        "row_number",
        "order_id",
        "errors"
    ]

    writer = csv.DictWriter(file, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerows(rejected_rows)

manifest_file = reports_folder / "ingestion_manifest.csv"

manifest_fields = [
    "file_name",
    "processed_at",
    "file_status",
    "rows_read",
    "rows_accepted",
    "rows_rejected",
    "error_message"
]

manifest_exists = manifest_file.exists()

with open(manifest_file, "a", newline="", encoding="utf-8") as file:

    writer = csv.DictWriter(file, fieldnames=manifest_fields)

    if not manifest_exists:
        writer.writeheader()

    writer.writerows(manifest_rows)

print("\nINGESTION SUMMARY")
print("Files processed:", len(manifest_rows))
print("Total accepted rows:", len(accepted_rows))
print("Total rejected rows:", len(rejected_rows))
print("Accepted output:", output_file)
print("Rejected report:", rejection_file)
print("Manifest:", manifest_file)
```

This script is a teaching implementation, not yet a production-grade concurrent ingestion service. We will review its design limitations below.

## 20.5 Understand the pipeline in sections

### Section A — Import modules

- csv reads and writes CSV data.
- shutil moves source files.
- Path represents filesystem locations.
- datetime records processing time.

### Section B — Prepare folders

The mkdir calls ensure that output locations exist. Without this step, opening a file inside a missing output directory raises FileNotFoundError.

### Section C — Define the contract

expected_headers defines the exact structure accepted by this classroom pipeline.

allowed_statuses defines the accepted business values.

A data contract is an agreement about what data is expected. It can include names, types, required fields, allowed values, semantics and delivery expectations.

### Section D — Discover files

glob("orders_*.csv") selects CSV files whose names begin with orders_.

sorted() ensures predictable processing order.

### Section E — Process one file at a time

The outer for loop handles one source file. Counters reset for every file so the manifest can report file-level totals.

### Section F — Check the file

A zero-byte file is rejected before CSV parsing.

The encoding utf-8-sig accepts ordinary UTF-8 and removes a UTF-8 byte-order mark if present. This is useful for some files exported by spreadsheet tools.

### Section G — Validate the header

DictReader uses header values as dictionary keys. We compare the actual header to the required header before reading records.

### Section H — Validate rows

Each row is checked for:
- Positive integer order ID.
- Non-empty customer name.
- Numeric, non-negative amount.
- Allowed status.

All errors for a row are collected. Invalid rows are written to a rejection report with the source file and row number.

### Section I — Archive or reject the file

A file with no file-level error is marked SUCCESS and moved to archive, unless the archive destination already exists.

A file with a file-level error is marked REJECTED and moved to rejected.

Notice the policy: row-level invalid records do not automatically reject the whole file. The valid rows can still be accepted. This is a business decision and must match the source contract.

### Section J — Write outputs

The accepted records go to accepted_orders.csv.

Rejected rows go to rejected_rows.csv.

Every file-level processing result is appended to ingestion_manifest.csv.

---

# 21. Expected project results

If the incoming directory contains:
- The three original valid files.
- The invalid test file.

Then the expected totals are:

| File | Rows read | Accepted | Rejected |
|---|---:|---:|---:|
| orders_mobile_20261110.csv | 3 | 3 | 0 |
| orders_partner_20261110.csv | 3 | 3 | 0 |
| orders_website_20261110.csv | 3 | 3 | 0 |
| orders_test_20261110.csv | 5 | 1 | 4 |
| **Total** | **14** | **10** | **4** |

The exact display order follows alphabetical file ordering.

The accepted output contains 10 records. The rejected report contains four invalid records.

The three original files and the test file are moved out of incoming after successful file processing. The test file has row-level rejections, but the file structure itself is valid, so it is processed and archived under the classroom policy.

## 21.1 Inspect accepted output

Open:

```text
data/output/accepted_orders.csv
```

It should contain the original nine valid records plus order 1010 from the test file.

The source_file column shows which input file supplied each accepted record.

## 21.2 Inspect rejected rows

Open:

```text
reports/rejected_rows.csv
```

It should contain four rejected records:
- Empty customer name.
- Non-numeric amount.
- Negative amount.
- Unsupported order status.

## 21.3 Inspect the manifest

Open:

```text
reports/ingestion_manifest.csv
```

It should contain one processing record per discovered input file.

The processed_at value changes with the system clock. The row counts and file statuses should match the table above.

---

# 22. Important limitations and production improvements

The project teaches the core flow, but students must understand what would need improvement before running a production pipeline.

## 22.1 The output file is overwritten

The project opens accepted_orders.csv in write mode. A new run replaces the previous output.

That is acceptable for a classroom demonstration, but not for a recurring production feed unless the output is intentionally a complete rebuild.

Production alternatives:
- Write one output file per batch.
- Append only after deduplication and validation.
- Load into a database staging table.
- Write immutable files to a date-partitioned object-storage path.

## 22.2 Re-running the project

After the first run, files have moved to archive or rejected. The incoming folder may be empty.

If the sample-file generator is run again, it recreates the original three files. The project output is overwritten, and the manifest appends new run records.

This is not a complete idempotency solution. It is a classroom demonstration of file movement and manifest recording.

A production system should use a stable file identity, processing state and duplicate policy.

## 22.3 File move and manifest write are not one transaction

The filesystem move and manifest append are separate operations. The process could stop after moving a file but before writing its manifest row.

Production systems need recovery logic:
- Reconcile incoming, processing, archive and rejected folders.
- Record processing state in a durable metadata store.
- Use atomic file moves within the same filesystem where supported.
- Make recovery safe after partial completion.
- Avoid marking a file complete until required outputs are durable.

## 22.4 Multiple workers

Two workers could discover the same file at the same time. The classroom script assumes one process.

A multi-worker design needs a safe claim mechanism, such as atomically moving a file into a worker-specific processing directory or using a database-backed work queue.

## 22.5 Very large files

The project uses accepted_rows and rejected_rows lists. This stores records in memory.

For very large files:
- Read one row at a time.
- Write accepted and rejected records incrementally.
- Process bounded batches.
- Avoid reading the entire file into memory.
- Monitor memory use.

You learned generators in Day 11. The same principle—processing data progressively—helps with large file ingestion.

## 22.6 Security considerations

- Do not execute files received from external sources.
- Restrict folder permissions.
- Validate file types and content rather than trusting extensions alone.
- Avoid logging personal or confidential data unnecessarily.
- Scan files according to company security policy.
- Do not accept arbitrary paths from untrusted input.
- Protect archived source data according to retention and access policies.

---

# 23. Common errors and troubleshooting

| Error or symptom | Possible reason | What to do |
|---|---|---|
| FileNotFoundError | Wrong path or file not delivered | Check current directory and file name |
| PermissionError | File is open or access is restricted | Close the file or check permissions |
| UnicodeDecodeError | File encoding differs from UTF-8 | Confirm source encoding; do not guess silently |
| JSONDecodeError | Invalid JSON syntax | Inspect the file and report parser position |
| KeyError | Expected column/key is missing | Validate schema before reading fields |
| ValueError during float conversion | Amount contains non-numeric text | Reject row and record reason |
| Empty output | No eligible files or no valid rows | Check discovery, filters and validation counts |
| Duplicate rows | File or record was processed more than once | Apply file identity and business-key checks |
| File moved but no manifest | Process stopped between operations | Reconcile folders and processing metadata |
| File processed too early | Source upload was incomplete | Require completion marker or atomic delivery |

## 23.1 FileNotFoundError example

Wrong:

```python
open("orders.csv", "r")
```

This only works if orders.csv is in the current working directory.

Better:

```python
from pathlib import Path

file_path = Path("data/incoming/orders_website_20261110.csv")

print("Exists:", file_path.exists())
```

Run the script from the project root or construct paths relative to the script location.

## 23.2 KeyError example

If the CSV has a column named customer instead of customer_name, this expression fails:

```python
row["customer_name"]
```

Check the header first and provide a clear schema error instead of allowing an unexplained KeyError.

## 23.3 ValueError example

This fails:

```python
amount = float("ten thousand")
```

Because the text is not a valid floating-point number.

Handle the conversion with try/except and record the rejected value and row.

---

# 24. Data Engineer design decisions

When asked to design a file ingestion system, do not start by saying "I will use pandas read_csv."

First clarify the operational requirements.

| Design question | Why it matters |
|---|---|
| Who sends the files? | Source ownership and delivery contract |
| How often do files arrive? | Schedule and freshness |
| How large are the files? | Memory and batch strategy |
| What naming convention is guaranteed? | Discovery and routing |
| How do we know upload is complete? | Prevent partial reads |
| Is the schema fixed? | Validation and schema evolution |
| Are partial valid rows allowed? | Reject policy |
| Can files be re-sent? | Duplicate handling |
| How long must raw files be retained? | Archive and compliance |
| What happens on failure? | Recovery and alerting |
| Is the data sensitive? | Security and access control |
| How do we reconcile counts? | Completeness evidence |

A Data Engineer designs the whole ingestion contract, not just the read statement.

---

# 25. Exercises with answers

## Exercise 1 — Find only CSV files

Folder contents:

```text
orders.csv
customers.json
readme.txt
payments.CSV
upload.tmp
```

Write a condition that selects CSV files regardless of extension case.

**Answer:**

```python
if file_path.suffix.lower() == ".csv":
    print(file_path.name)
```

## Exercise 2 — Count all rows

There are four files. Each has 100 data rows.

How many data rows should the pipeline read if every file is valid?

**Answer:** 400 rows. Do not count the header row as a data record.

## Exercise 3 — Identify a file-level error

A file contains valid CSV data but is missing the amount column.

Should the pipeline process it?

**Answer:** Not under the stated contract. The file schema is invalid. Reject the file or route it to a controlled schema-change process.

## Exercise 4 — Identify row-level errors

A file has the correct header and 1,000 records. Three rows contain invalid amounts.

What are the accepted and rejected counts if partial acceptance is allowed?

**Answer:** 997 accepted and 3 rejected.

## Exercise 5 — Why archive files?

**Answer:** Archiving preserves source evidence, helps investigate previous runs and supports controlled reprocessing.

## Exercise 6 — Why is a file name not a unique identity?

**Answer:** The same file name can be reused for different contents, and identical content can arrive under different names. A stable delivery ID or content fingerprint can help identify files.

## Exercise 7 — How do you prevent partial file reads?

**Answer:** Prefer a source delivery contract where the source writes a temporary file and atomically renames it after completion, or provides a completion marker. File-size stability checks are supplementary, not definitive proof.

## Exercise 8 — Why keep a manifest?

**Answer:** It records which files were processed, when, with what status and how many records were accepted or rejected. It supports audit, troubleshooting and operational reporting.

## Exercise 9 — What is the difference between file rejection and row rejection?

**Answer:** File rejection means the file itself is unusable or violates the file contract. Row rejection means the file is readable but one or more individual records violate data rules.

## Exercise 10 — What happens if the script fails after moving a file but before writing the manifest?

**Answer:** The file may be archived without a corresponding manifest record. A production system needs reconciliation and recoverable processing state to repair this gap.

---

# 26. Student practical assignment

**Project:** Multi-File Ingestion System

Build the project without copying the entire solution blindly. Implement each stage and run it before moving to the next.

## Required folder structure

```text
day31_multi_file_ingestion/
|
|-- data/
|   |-- incoming/
|   |-- archive/
|   |-- rejected/
|   |-- output/
|
|-- reports/
|
|-- 01_create_sample_files.py
|-- 02_scan_directory.py
|-- 03_read_multiple_csv.py
|-- 04_validate_files.py
|-- 05_ingestion_pipeline.py
|-- README.md
```

## Required behavior

1. Create three valid CSV source files.
2. Discover all eligible files in incoming.
3. Ignore unrelated extensions and temporary files.
4. Check the expected file naming convention.
5. Check required headers.
6. Read all valid files.
7. Validate order IDs, customer names, amounts and statuses.
8. Keep accepted records.
9. Write invalid records and rejection reasons.
10. Archive successfully processed files.
11. Move file-level failures to rejected.
12. Write an ingestion manifest.
13. Print summary counts.
14. Explain how the project should be changed for large files and multiple workers.

## Submission evidence

Students should provide:
- Screenshot or text output of discovered files.
- Sample source files.
- Accepted output file.
- Rejected row report.
- Ingestion manifest.
- Folder state after processing.
- A short explanation of validation rules.
- A short explanation of how reprocessing is controlled.

## Final validation checklist

- [ ] Project folders are created automatically.
- [ ] Only expected files are discovered.
- [ ] Temporary files are ignored.
- [ ] File names are checked.
- [ ] Empty files are detected.
- [ ] CSV headers are validated.
- [ ] CSV records are read correctly.
- [ ] Numeric values are converted safely.
- [ ] Invalid rows include rejection reasons.
- [ ] Accepted and rejected counts are correct.
- [ ] Successful files are archived.
- [ ] File-level failures are isolated.
- [ ] A manifest is created or updated.
- [ ] The output includes source file lineage.
- [ ] The student can explain production limitations.

---

# 27. Day 31 revision notes

| Concept | Simple explanation |
|---|---|
| File ingestion | Bringing file data into a data platform |
| Directory scanning | Finding files in a folder |
| pathlib | Python module for filesystem paths |
| iterdir | Lists direct directory entries |
| glob | Finds paths matching a pattern |
| rglob | Searches matching paths recursively |
| suffix | File extension |
| stem | File name without extension |
| CSV reader | Reads CSV rows as lists |
| DictReader | Reads CSV rows as dictionaries |
| JSON load | Reads JSON from an open file |
| File-level validation | Checks whether the complete file is acceptable |
| Row-level validation | Checks individual records |
| Archive | Stores successfully processed source files |
| Rejected folder | Isolates files that cannot be accepted |
| Manifest | Tracks file processing status and counts |
| File fingerprint | Hash representing file content |
| Idempotency | Repeating a pipeline should not create unintended effects |
| Lineage | Ability to trace data back to its source |

## The five rules to remember

1. Never read every file blindly. Discover and filter files using an agreed naming and delivery contract.
2. Validate the file structure before trusting its records.
3. Validate individual records and record why a row was rejected.
4. Preserve source files and processing evidence.
5. Design for retries, duplicate deliveries and partial failures.

**Today's key lesson:** A reliable file-ingestion pipeline does not simply read files. It discovers, validates, processes, records and safely manages them.

**Next class:** Day 32 — API-Based Data Ingestion. You will learn how a pipeline collects data from REST APIs, handles pagination, authentication, rate limits, retries and incremental API extraction.
