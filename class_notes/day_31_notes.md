# Day 31: File-Based Data Ingestion
**Phase:** 4 — Data Processing and Pipelines  
**Learning time:** 2–3 hours  
**Audience:** Complete beginners and freshers  
**Topic boundary:** This file teaches file-based ingestion only. The separate project workbook and interview guide are not included here.

> **How to use this lesson:** Read one section, run its code in order, compare your output, and explain the result in your own words. Do not skip setup or validation steps. A file being readable does not mean its data is trustworthy.

## Learning outcomes

By the end of this class, you should be able to:
- explain what file-based ingestion means and why businesses use it;
- distinguish a file, record, field, header, delimiter, encoding and schema;
- create and inspect a practice folder on Windows;
- read CSV files using Python's built-in `csv` module;
- understand when to use `csv.reader`, `csv.DictReader` and Pandas;
- read JSON objects and arrays;
- validate filenames, headers, rows, types, required values and business rules;
- handle missing, empty, malformed and duplicate files;
- preserve rejected records with reasons instead of silently dropping them;
- write accepted and rejected outputs safely;
- avoid processing the same delivery twice;
- reconcile counts and explain what the checks prove;
- recognize the limitations of local file ingestion and the controls needed in production.

## Today's learning route

`Business request -> File contract -> Folder setup -> Discover -> Inspect -> Read -> Validate -> Accept/reject -> Write -> Reconcile -> Archive`

---

# 1. Start with the business problem

Imagine TechStore sells products online. It receives data from several outside partners.

- A payment company sends a CSV file containing payment settlements.
- A courier company sends a JSON file containing delivery updates.
- A supplier sends a CSV file containing inventory levels.
- A marketing company sends campaign performance files.

These partners may not have direct access to TechStore's database. Sending a file is a simple way to exchange information.

## 1.1 What is file-based data ingestion?

**File-based data ingestion** is the controlled process of receiving data stored in files, reading the file, checking whether its contents follow agreed rules, and making the valid data available to another system.

Break the phrase into parts:

| Word | Beginner meaning | Example |
|---|---|---|
| File | A named item stored on disk | `partner_orders.csv` |
| Data | Facts represented inside the file | Order ID, amount, date |
| Ingestion | Bringing data into a system under controlled steps | Reading and validating partner orders |
| Source | Where the data comes from | Courier company |
| Destination | Where accepted data is stored | Analytics database |
| Record | One business item, usually one row | One order |
| Field | One piece of information in a record | `order_id` |
| Header | The names of the fields | First CSV row |
| Schema | The agreed structure and types | Order ID is text; amount is decimal |

A file ingestion pipeline does more than open a file. It must answer:
1. Did the expected file arrive?
2. Is it the correct file?
3. Can the file be decoded?
4. Does it contain the required columns?
5. Are the records valid?
6. Were all records accounted for?
7. Was the output written successfully?
8. Can we prove what happened?

## 1.2 File ingestion is not the same as copying a file

Suppose a partner sends 1,000 order records.

A simple copy operation moves the file from one folder to another. It does not prove that:
- all 1,000 records can be read;
- the amount column contains valid numbers;
- order IDs are unique;
- required fields are present;
- the file belongs to the correct business date;
- the output contains all valid rows.

A data engineer must create checks around the file.

## 1.3 Example of a business request

The operations manager says:

> Every morning, process the partner order file. Keep valid orders, send invalid orders to a separate rejected file, and report how many records were received, accepted and rejected.

Translate the request into technical requirements:

| Business sentence | What the engineer must implement |
|---|---|
| Process the partner order file | Discover the expected file in an input folder |
| Every morning | Define a schedule outside this lesson |
| Keep valid orders | Apply agreed validation rules |
| Send invalid orders separately | Write rejected records with rejection reasons |
| Report counts | Count input, accepted and rejected records |
| Process the partner file | Record the source filename and processing result |

**Important:** The manager has not told us what makes an order valid. We must ask the data owner and document the rules. A developer must not invent business rules.

## 1.4 What happens if we skip validation?

Imagine the file contains these records:

| partner_order_id | customer_id | amount | status |
|---|---:|---:|---|
| P-1001 | 501 | 1250.50 | PAID |
| P-1002 | 502 | 800.00 | SHIPPED |
| P-1003 | 503 | eight hundred | PAID |
| P-1004 | blank | 400.00 | PAID |

The third row has text where a numeric amount is expected. The fourth row has no customer ID.

If the pipeline sends every row directly to a database, the load may fail or incorrect data may enter the system.

A controlled pipeline should:
- accept records that satisfy the documented rules;
- reject or quarantine records that do not;
- retain the original values for investigation;
- explain why each record was rejected;
- reconcile the number of input records.

---

# 2. Understand the file contract before writing code

A **file contract** is an agreement between the file producer and the file consumer. It defines what a valid delivery looks like.

## 2.1 Example contract for partner orders

| Property | Contract |
|---|---|
| Producer | Partner order system |
| Consumer | TechStore data platform |
| Format | CSV |
| Encoding | UTF-8 |
| Delimiter | Comma |
| Header | Required |
| File pattern | `partner_orders_YYYY-MM-DD.csv` |
| Required fields | partner_order_id, customer_id, order_date, status, amount |
| Unique identity | partner_order_id within the partner |
| Amount | Decimal number, zero or greater |
| Date | ISO format YYYY-MM-DD |
| Status | PAID, SHIPPED, PENDING, CANCELLED |
| Empty delivery | Must be explicitly agreed |
| Duplicate delivery | Must not create duplicate business records |

Do not assume every partner follows this contract. Each source can have its own contract.

## 2.2 Why should we document the producer?

If the file is wrong, we need to know who can explain it.

For example:
- The partner owns the source values.
- TechStore owns the rules for accepting those values.
- The data engineer owns the technical ingestion implementation.
- The operations team may own the delivery schedule.

Clear ownership prevents teams from guessing.

## 2.3 Required and optional fields

A **required field** must contain a usable value for a record to be accepted.

An **optional field** may be empty when the contract permits it.

Example:

| Field | Required? | Reason |
|---|---|---|
| partner_order_id | Yes | Identifies the partner order |
| customer_id | Yes | Connects the order to a customer |
| order_date | Yes | Needed for business reporting |
| status | Yes | Needed to understand order state |
| amount | Yes | Needed for financial reporting |
| coupon_code | No | An order may not use a coupon |

A missing optional coupon code is not automatically an invalid record. A missing required order ID normally is.

## 2.4 Schema and data values are different

The **schema** describes the structure:
- field names;
- expected data types;
- required or optional status;
- sometimes allowed values and constraints.

The **data values** are the actual contents.

Example:
- Schema says `amount` should be decimal.
- Value `1250.50` follows that expectation.
- Value `one thousand` does not.

A file can have the correct columns but still contain invalid values.

## 2.5 Contract changes must be agreed

Suppose the partner renames `customer_id` to `buyer_id`.

Do not silently change the pipeline and assume the two fields mean the same thing.

Ask:
1. Is this a permanent rename?
2. Does the new field have the same meaning?
3. Will both names appear during a transition?
4. From which delivery date does the change apply?
5. Who approved the change?
6. Should old files still be accepted?

A schema change is a communication and compatibility problem, not merely a Python problem.

---

# 3. Understand CSV files from zero

CSV means **Comma-Separated Values**. It is a text format commonly used to represent rows and columns.

## 3.1 Look at a tiny CSV file

Create a file named `orders.csv` containing:

```csv
order_id,customer_id,amount,status
O-101,501,1250.50,PAID
O-102,502,800.00,SHIPPED
O-103,503,250.00,PENDING
```

Read it as a human:
- The first line contains column names.
- The second line represents the first order.
- The third line represents the second order.
- The fourth line represents the third order.

There are four lines in this simple file: one header line and three data lines.

## 3.2 A CSV file is text, not a database table

The value `1250.50` is stored as characters in a text file. CSV does not automatically guarantee that it is a decimal number.

When Python reads a CSV value, it often receives text first. We must convert and validate it.

For example:
- `"1250.50"` is text.
- `1250.50` can be a numeric decimal value after conversion.
- `"eight hundred"` cannot be converted to a decimal.

## 3.3 What is a delimiter?

A delimiter separates fields.

Common delimiters:
- comma: `,`;
- tab: a tab character;
- semicolon: `;`;
- pipe: `|`.

A file may be called CSV even when a regional system uses a semicolon delimiter. Always confirm the contract.

Example using a pipe:

```text
order_id|customer_id|amount
O-101|501|1250.50
O-102|502|800.00
```

If Python expects commas but the file uses pipes, the whole line may be read as one field. That is a parsing/configuration problem.

## 3.4 Why quotation marks matter

A field can contain a comma as part of its value.

```csv
customer_id,customer_name,city
501,"Ravi, Kumar",Hyderabad
502,"Anita",Chennai
```

The comma inside `"Ravi, Kumar"` belongs to the name. It does not separate two columns.

Do not split CSV lines using a simple string `split(",")` in production. It fails for quoted commas, escaped quotes and other valid CSV cases.

## 3.5 Newlines inside quoted fields

Some CSV files allow a text field to contain a line break.

This means one logical record can occupy more than one physical text line.

A proper CSV parser understands quoting rules. Counting newline characters is not always the same as counting records.

## 3.6 Header row

A header gives meaning to each position.

Without a header, a reader may know that a row has five values but not which value represents the order ID.

A file contract must say whether a header exists.

## 3.7 Empty string and missing field

These are not always identical.

Example:

```csv
order_id,customer_id,amount
O-101,501,100
O-102,,200
O-103,503
```

- In row O-102, customer_id is present as an empty field.
- In row O-103, the final amount field is absent.
- A parser may represent both situations differently depending on how the row is read.

The ingestion code should check row length and field content separately.

---

# 4. Understand JSON files from zero

JSON means **JavaScript Object Notation**. It is a text format used to represent objects, arrays and values.

## 4.1 A JSON object

```json
{
  "order_id": "O-101",
  "customer_id": 501,
  "amount": 1250.50,
  "status": "PAID"
}
```

This represents one order object.

Important parts:
- Curly braces represent an object.
- A key identifies a property.
- A colon separates a key from its value.
- A comma separates properties.
- Text values use double quotes.
- Numeric values do not need quotes.

## 4.2 A JSON array

```json
[
  {
    "order_id": "O-101",
    "customer_id": 501,
    "amount": 1250.50
  },
  {
    "order_id": "O-102",
    "customer_id": 502,
    "amount": 800.00
  }
]
```

The square brackets represent a list of two objects.

A common beginner mistake is to expect every JSON file to contain an array. A file can contain one object, an array, or a nested structure. Inspect the actual contract.

## 4.3 JSON values map to Python values

| JSON value | Python value |
|---|---|
| Object | Dictionary |
| Array | List |
| String | String |
| Number | Integer or float |
| true | True |
| false | False |
| null | None |

Python uses capitalized `True`, `False` and `None`. JSON uses lowercase `true`, `false` and `null`.

## 4.4 CSV or JSON: which one should we choose?

| Requirement | CSV | JSON |
|---|---|---|
| Simple rows and columns | Very suitable | Possible |
| Nested object structure | Awkward | Natural |
| Human inspection | Easy for tabular data | Easy for structured objects |
| Data types | Mostly text values | Supports basic JSON types |
| Large tabular exchange | Common | Can be larger |
| Repeating nested child records | Awkward | Natural |

Neither format is universally better. Choose based on the producer, consumer, schema and use case.

---

# 5. Prepare the Windows learning environment

We will use Python's built-in libraries first. No external package is required for the basic CSV and JSON examples.

## 5.1 Open Windows Command Prompt

1. Press the Windows key.
2. Type `cmd`.
3. Open Command Prompt.
4. You should see a prompt similar to `C:\Users\Student>`.

Do not type Python code directly at the Windows command prompt. CMD commands and Python statements are different languages.

## 5.2 Create a learning folder

Run these commands one at a time:

```bat
mkdir day31_file_ingestion
cd day31_file_ingestion
mkdir input
mkdir accepted
mkdir rejected
mkdir archive
mkdir reports
```

What each command means:

| Command | Meaning |
|---|---|
| `mkdir` | Create a directory |
| `cd` | Change the current directory |
| `input` | Original files waiting for processing |
| `accepted` | Valid output records |
| `rejected` | Invalid records and their reasons |
| `archive` | Original files retained after processing |
| `reports` | Run summaries and reconciliation evidence |

After the commands, your folder should look like:

```text
day31_file_ingestion/
    input/
    accepted/
    rejected/
    archive/
    reports/
```

## 5.3 Confirm your current directory

Run:

```bat
cd
```

Expected output: the full path of the current folder. It will vary by computer.

Why do we check this? Relative paths are interpreted from the current working directory. If you run the notebook from another folder, a path that worked yesterday may not work today.

## 5.4 Open the folder in VS Code

If VS Code is installed, run:

```bat
code .
```

The dot means the current directory.

If the command is not recognized:
1. Open VS Code from the Start menu.
2. Select File.
3. Select Open Folder.
4. Choose `day31_file_ingestion`.
5. Confirm the folder name in the Explorer panel.

## 5.5 Create a Python file for the first test

Create a file named `check_environment.py` in the main folder.

Enter:

```python
print("Python is ready")
```

Run from CMD:

```bat
python check_environment.py
```

Expected output:

```text
Python is ready
```

Explanation:
- `print` is a Python built-in function.
- Parentheses contain the value to display.
- Text is written inside quotation marks.
- Python executes the statement and displays the text.

If CMD says Python is not recognized, Python may not be installed or may not be available on PATH. Fix the installation before continuing.

## 5.6 Create the sample CSV file

Inside the `input` folder, create `partner_orders_2026-11-10.csv`.

Enter exactly:

```csv
partner_order_id,customer_id,order_date,status,amount
P-1001,501,2026-11-10,PAID,1250.50
P-1002,502,2026-11-10,SHIPPED,800.00
P-1003,503,2026-11-10,PENDING,250.00
P-1004,504,2026-11-10,CANCELLED,0.00
```

Save as UTF-8. Make sure the filename ends with `.csv`, not `.csv.txt`.

---

# 6. First Python inspection: understand paths

Before reading file contents, Python must locate the file.

## 6.1 Import Path

```python
from pathlib import Path
```

Explanation:
- `pathlib` is a standard Python module for working with filesystem paths.
- `Path` is a class provided by that module.
- `from ... import ...` makes the named object available directly.
- We use it because path handling is clearer than manually joining strings.

## 6.2 Point to the input folder

Run this from a script located in the main project folder:

```python
from pathlib import Path

input_folder = Path("input")
print(input_folder)
print(input_folder.exists())
```

Expected output:

```text
input
True
```

Explanation:
- `Path("input")` creates a path object pointing to the input folder relative to the current working directory.
- `input_folder` stores that path.
- `print(input_folder)` displays the path representation.
- `exists()` checks whether the path exists.
- `True` means the folder exists at that location.

If the result is False, check the current working directory and folder spelling.

## 6.3 Check that the path is a directory

```python
print(input_folder.is_dir())
```

Expected output:

```text
True
```

`is_dir()` asks whether the path points to a directory. A path can exist and still point to a file, so these checks answer different questions.

## 6.4 Check one exact file

```python
file_path = input_folder / "partner_orders_2026-11-10.csv"

print(file_path)
print(file_path.exists())
print(file_path.is_file())
```

Expected output:

```text
input/partner_orders_2026-11-10.csv
True
True
```

The slash operator on a Path object joins path components. It is safer than manually inserting slash characters because pathlib handles operating-system path rules.

## 6.5 Absolute and relative paths

A relative path is interpreted from the current working directory.

Example:
`input/partner_orders.csv`

An absolute path identifies a location from the filesystem root.

Example:
`D:\Data\day31_file_ingestion\input\partner_orders.csv`

Use relative paths for portable learning folders. Use configured absolute paths when an application has a fixed deployment location.

## 6.6 Common path error

Error:

```text
FileNotFoundError: [Errno 2] No such file or directory
```

Possible causes:
- wrong filename;
- wrong extension;
- wrong folder;
- notebook launched from a different working directory;
- file has not arrived;
- spelling or capitalization differs on a case-sensitive filesystem.

Debug in this order:
1. Print the current working directory.
2. Check the parent folder exists.
3. List the folder contents.
4. Copy the exact filename.
5. Check the file extension.
6. Retry only after correcting the path.

---

# 7. Discover incoming files

A pipeline should not assume that a specific file exists. It should discover files according to a contract.

## 7.1 List everything in the input folder

```python
from pathlib import Path

input_folder = Path("input")
items = list(input_folder.iterdir())

print(items)
```

Expected output contains the sample file path. The exact display depends on the operating system.

Line-by-line:
- `input_folder.iterdir()` creates an iterator over immediate entries.
- `list(...)` converts the iterator to a list so we can display and count it.
- `items` stores the discovered entries.
- `print(items)` displays the entries.

The list may contain files and subfolders. We should not assume every entry is a valid delivery.

## 7.2 Discover only CSV files

```python
csv_files = list(input_folder.glob("*.csv"))

print("CSV files found:", len(csv_files))
for file_path in csv_files:
    print(file_path.name)
```

Expected output:

```text
CSV files found: 1
partner_orders_2026-11-10.csv
```

Explanation:
- `glob("*.csv")` selects names matching the CSV pattern.
- The asterisk means any sequence of characters.
- `.csv` requires the filename to end with that extension.
- `len(csv_files)` counts the discovered paths.
- The `for` loop visits each path.
- `file_path.name` displays only the filename, not the full path.

## 7.3 Why should we not process every file blindly?

The folder might contain:
- a temporary upload;
- a text explanation;
- an old file;
- a file from another partner;
- a partially uploaded file;
- an unrelated CSV.

A robust pipeline filters by source, naming convention, date, extension and delivery status.

## 7.4 Check for an empty folder

```python
csv_files = list(input_folder.glob("*.csv"))

if len(csv_files) == 0:
    print("No CSV files found")
else:
    print("CSV files are available")
```

Expected output when the sample exists:

```text
CSV files are available
```

If the folder is empty, the output is:

```text
No CSV files found
```

An empty folder is not always a technical failure. It depends on the delivery agreement:
- If no file is expected today, it may be normal.
- If a daily file was due at 07:00, it may be a missing-delivery incident.

The business schedule determines the meaning.

## 7.5 Sort discovered files

```python
csv_files = sorted(input_folder.glob("*.csv"))

for file_path in csv_files:
    print(file_path.name)
```

Sorting gives a predictable order. It does not prove that the files should be processed in that order. If ordering matters, define the rule explicitly, such as business date or sequence number.

## 7.6 Filter by a filename pattern

```python
csv_files = list(input_folder.glob("partner_orders_*.csv"))

for file_path in csv_files:
    print(file_path.name)
```

This selects only files beginning with `partner_orders_` and ending in `.csv`.

Do not rely on a broad `*.csv` pattern if the same folder receives different feeds.

## 7.7 Inspect file size

```python
for file_path in csv_files:
    print(file_path.name, file_path.stat().st_size)
```

`stat()` returns file metadata. `st_size` is the file size in bytes.

A zero-byte file may indicate an empty or incomplete delivery. But a non-zero size does not prove the file is valid.

---

# 8. Read CSV with Python's csv module

Python includes a standard library module named `csv`. We can use it without installing another package.

## 8.1 Open a file safely

```python
import csv
from pathlib import Path

file_path = Path("input/partner_orders_2026-11-10.csv")

with file_path.open("r", encoding="utf-8", newline="") as file:
    print(file.read())
```

Expected output is the exact CSV text that you saved.

Explanation:
- `import csv` loads the CSV module.
- `Path(...)` identifies the file.
- `open("r")` opens the file in read mode.
- `encoding="utf-8"` tells Python how to decode the text.
- `newline=""` is recommended when using the CSV module so it can handle newline conventions correctly.
- `with` manages the file resource.
- When the indented block ends, Python closes the file automatically.
- `read()` reads the entire file as one text string.

**Memory warning:** `read()` loads the entire file into memory. It is fine for this tiny demonstration, but not a good choice for a very large file.

## 8.2 What is a file handle?

The variable `file` is a file object. It represents the open file and allows Python to read from it.

A file object has a current position. After reading to the end, another read may return an empty string unless we seek back to the beginning or reopen the file.

## 8.3 Read one line

```python
with file_path.open("r", encoding="utf-8", newline="") as file:
    first_line = file.readline()
    print(first_line)
```

Expected output:

```text
partner_order_id,customer_id,order_date,status,amount
```

`readline()` reads one physical line. Remember that quoted CSV fields can contain line breaks, so a physical line is not always a complete logical record.

## 8.4 Read all lines

```python
with file_path.open("r", encoding="utf-8", newline="") as file:
    lines = file.readlines()

print(len(lines))
```

Expected output for the four-row sample including the header:

```text
5
```

This simple sample has five physical lines. For general CSV record counting, use the CSV parser rather than counting lines.

## 8.5 Use csv.reader

```python
import csv

with file_path.open("r", encoding="utf-8", newline="") as file:
    reader = csv.reader(file)

    for row in reader:
        print(row)
```

Expected output:

```text
['partner_order_id', 'customer_id', 'order_date', 'status', 'amount']
['P-1001', '501', '2026-11-10', 'PAID', '1250.50']
['P-1002', '502', '2026-11-10', 'SHIPPED', '800.00']
['P-1003', '503', '2026-11-10', 'PENDING', '250.00']
['P-1004', '504', '2026-11-10', 'CANCELLED', '0.00']
```

Notice that the values are strings. The parser separates fields; it does not automatically apply the business data types.

## 8.6 Understand the reader object

- `csv.reader(file)` creates a reader.
- The reader returns one row at a time.
- Each row is represented as a list.
- Each list item is a field value.
- The `for` loop asks for the next row until the file is exhausted.

This row-by-row behavior is useful because we do not need to create one huge list of all records.

## 8.7 Read the header separately

```python
with file_path.open("r", encoding="utf-8", newline="") as file:
    reader = csv.reader(file)
    header = next(reader)

print(header)
```

Expected output:

```text
['partner_order_id', 'customer_id', 'order_date', 'status', 'amount']
```

`next(reader)` requests the next item from the iterator. The first item is the header row in this contract.

If the file is empty, there is no first item. Handle that situation rather than allowing an unexplained StopIteration error.

## 8.8 Use csv.DictReader

A dictionary reader uses the header names as keys.

```python
with file_path.open("r", encoding="utf-8", newline="") as file:
    reader = csv.DictReader(file)

    for row in reader:
        print(row)
```

Expected output:

```text
{'partner_order_id': 'P-1001', 'customer_id': '501', 'order_date': '2026-11-10', 'status': 'PAID', 'amount': '1250.50'}
{'partner_order_id': 'P-1002', 'customer_id': '502', 'order_date': '2026-11-10', 'status': 'SHIPPED', 'amount': '800.00'}
{'partner_order_id': 'P-1003', 'customer_id': '503', 'order_date': '2026-11-10', 'status': 'PENDING', 'amount': '250.00'}
{'partner_order_id': 'P-1004', 'customer_id': '504', 'order_date': '2026-11-10', 'status': 'CANCELLED', 'amount': '0.00'}
```

A dictionary makes the code easier to read:
- `row["partner_order_id"]` is clearer than `row[0]`.
- `row["amount"]` is clearer than `row[4]`.

## 8.9 Compare reader and DictReader

| Tool | Row representation | Access |
|---|---|---|
| `csv.reader` | List | `row[0]` |
| `csv.DictReader` | Dictionary | `row["order_id"]` |

Use `csv.reader` when position-based processing is appropriate. Use `DictReader` when named fields improve clarity.

---

# 9. Understand CSV parsing options

## 9.1 Specify a semicolon delimiter

Create a file with:

```text
order_id;customer_id;amount
O-101;501;1250.50
O-102;502;800.00
```

Read it:

```python
with open("input/semicolon_orders.csv", "r", encoding="utf-8", newline="") as file:
    reader = csv.DictReader(file, delimiter=";")

    for row in reader:
        print(row)
```

Expected output:

```text
{'order_id': 'O-101', 'customer_id': '501', 'amount': '1250.50'}
{'order_id': 'O-102', 'customer_id': '502', 'amount': '800.00'}
```

The delimiter must match the file. If you use the default comma delimiter, the header may be treated as one field.

## 9.2 Handle quoted values

Create:

```csv
customer_id,customer_name,city
501,"Ravi, Kumar",Hyderabad
502,Anita,Chennai
```

Read with `csv.DictReader`. The first customer name should remain one value: `Ravi, Kumar`.

Do not manually split the line on commas.

## 9.3 Understand quote characters

CSV commonly uses a double quote to enclose a field that contains a delimiter or line break.

The CSV parser handles standard quoting rules. If a partner uses a non-standard escape convention, document it and configure the parser explicitly.

## 9.4 Detect a malformed row length

A row may have fewer or more fields than the header.

```python
with file_path.open("r", encoding="utf-8", newline="") as file:
    reader = csv.reader(file)
    header = next(reader)

    for row_number, row in enumerate(reader, start=2):
        if len(row) != len(header):
            print("Bad row:", row_number, row)
        else:
            print("Valid field count:", row_number)
```

Expected output for the clean sample:

```text
Valid field count: 2
Valid field count: 3
Valid field count: 4
Valid field count: 5
```

The row number starts at 2 because row 1 is the header.

A matching field count does not prove the values are valid. It proves only that the number of parsed fields matches the header count.

---

# 10. Validate the file before processing records

Validation should happen in layers. A malformed file structure may make record-level processing unsafe.

## 10.1 Layer 1 — Does the file exist?

```python
if not file_path.exists():
    print("File is missing")
else:
    print("File exists")
```

A missing file should be recorded clearly. Do not create an empty success output and pretend that the expected delivery was processed.

## 10.2 Layer 2 — Is it a regular file?

```python
if file_path.is_file():
    print("Path points to a file")
else:
    print("Path does not point to a regular file")
```

A directory can exist at a path where a file was expected. Check the path type.

## 10.3 Layer 3 — Is it empty?

```python
if file_path.stat().st_size == 0:
    print("File is empty")
else:
    print("File contains bytes")
```

A zero-byte file has no content. A file containing only a header has bytes but zero data records. These are different conditions.

## 10.4 Layer 4 — Can it be decoded?

A UTF-8 decoding error means the bytes do not form valid UTF-8 text under the chosen decoding rules.

Do not automatically switch to another encoding without agreement. Incorrect decoding can silently corrupt names and other values.

## 10.5 Layer 5 — Is the header present?

```python
required_columns = [
    "partner_order_id",
    "customer_id",
    "order_date",
    "status",
    "amount"
]

with file_path.open("r", encoding="utf-8", newline="") as file:
    reader = csv.reader(file)
    received_header = next(reader, None)

print(received_header)
```

The optional default passed to `next` means an empty file returns `None` instead of raising StopIteration.

## 10.6 Compare required and received columns

```python
if received_header is None:
    print("File has no header")
else:
    missing_columns = set(required_columns) - set(received_header)
    extra_columns = set(received_header) - set(required_columns)

    print("Missing:", missing_columns)
    print("Extra:", extra_columns)
```

Expected output for the correct sample:

```text
Missing: set()
Extra: set()
```

Explanation:
- `set(...)` creates a set of unique values.
- The minus operator finds values present in the left set but not in the right set.
- Missing columns are required fields not received.
- Extra columns are fields received but not included in the current contract.

Whether extra columns are allowed is a contract decision. Some pipelines allow them; strict pipelines stop when the schema changes.

## 10.7 Header order can change

These two headers contain the same names in different orders:

```text
order_id,customer_id,amount
customer_id,amount,order_id
```

With DictReader, values are mapped using header names. If the contract permits order changes, compare sets of names rather than requiring exact list order.

If downstream processing requires a specific order, normalize the output order explicitly.

## 10.8 Detect duplicate header names

A duplicate header can create ambiguous mappings.

```python
if received_header is not None:
    duplicate_headers = []

    for column_name in received_header:
        if received_header.count(column_name) > 1:
            if column_name not in duplicate_headers:
                duplicate_headers.append(column_name)

    print(duplicate_headers)
```

Expected output for a clean file:

```text
[]
```

This simple method is easy to understand but compares each name repeatedly. For very wide files, a frequency counter is more efficient.

## 10.9 Normalize header whitespace carefully

A partner may send `" amount "` rather than `"amount"`.

You may trim whitespace if the contract permits it:

```python
normalized_header = [name.strip() for name in received_header]
print(normalized_header)
```

Do not silently rename unrelated fields. Trimming accidental whitespace is a narrowly defined normalization rule; changing field meaning requires approval.

---

# 11. Convert text values into useful Python types

CSV values are commonly read as strings. Data engineering requires explicit conversion.

## 11.1 Convert customer ID to integer

```python
customer_id_text = "501"
customer_id = int(customer_id_text)

print(customer_id)
print(type(customer_id))
```

Expected output:

```text
501
<class 'int'>
```

Explanation:
- `customer_id_text` stores the original text.
- `int(...)` converts a valid integer string to an integer.
- `type(...)` shows the Python type.

Do not convert an identifier to an integer automatically if leading zeros are meaningful. A customer code such as `"000501"` may need to remain text.

## 11.2 Understand conversion failure

```python
amount_text = "eight hundred"
amount = float(amount_text)
```

This raises a ValueError because the text is not a valid floating-point representation.

A production pipeline must handle invalid input explicitly rather than crashing without recording which record caused the problem.

## 11.3 Use Decimal for money

Binary floating-point can represent many decimal fractions approximately. For money, Python's `Decimal` is often a better choice.

```python
from decimal import Decimal

amount_text = "1250.50"
amount = Decimal(amount_text)

print(amount)
print(type(amount))
```

Expected output:

```text
1250.50
<class 'decimal.Decimal'>
```

Construct Decimal from the original text. Avoid first converting the text to float and then passing the float into Decimal.

## 11.4 Convert a date

```python
from datetime import date

order_date_text = "2026-11-10"
order_date = date.fromisoformat(order_date_text)

print(order_date)
print(type(order_date))
```

Expected output:

```text
2026-11-10
<class 'datetime.date'>
```

If the date text is not in the expected ISO format, conversion raises an error. That record should be rejected or handled according to the contract.

## 11.5 Standardize status values

A partner might send `paid`, `PAID` or ` Paid `.

A controlled normalization may be:

```python
status_text = " paid "
status = status_text.strip().upper()

print(status)
```

Expected output:

```text
PAID
```

- `strip()` removes leading and trailing whitespace.
- `upper()` changes letters to uppercase.
- The rule should be documented so the same input always produces the same result.

Do not convert unknown values to PAID or another default. Unknown business values should be rejected or reviewed.

---

# 12. Validate one record step by step

We will now validate a single record before processing a complete file.

## 12.1 Define one sample record

```python
row = {
    "partner_order_id": "P-1001",
    "customer_id": "501",
    "order_date": "2026-11-10",
    "status": "PAID",
    "amount": "1250.50"
}
```

This dictionary represents one row from the partner file.

## 12.2 Check required values

```python
required_fields = [
    "partner_order_id",
    "customer_id",
    "order_date",
    "status",
    "amount"
]

missing_fields = []

for field in required_fields:
    if row.get(field) is None or row.get(field).strip() == "":
        missing_fields.append(field)

print(missing_fields)
```

Expected output:

```text
[]
```

Explanation:
- `required_fields` lists the required names.
- `missing_fields` starts empty.
- The loop checks each required field.
- `get(field)` retrieves a value without raising KeyError when the key is missing.
- The condition checks both a missing value and an empty/whitespace-only string.
- If a value is missing, its field name is appended.
- An empty result means no required field was found missing.

## 12.3 Why use get?

`row["customer_id"]` raises KeyError if the key does not exist.

`row.get("customer_id")` returns None when the key does not exist, unless another default is supplied.

This helps validation code inspect missing keys safely.

## 12.4 Validate the status against an allowed list

```python
allowed_statuses = ["PAID", "SHIPPED", "PENDING", "CANCELLED"]

status = row["status"].strip().upper()

if status in allowed_statuses:
    print("Status is accepted")
else:
    print("Status is not allowed")
```

Expected output:

```text
Status is accepted
```

The `in` operator checks membership. The allowed values must come from the business contract.

## 12.5 Validate the amount

```python
from decimal import Decimal, InvalidOperation

amount_text = row["amount"]

try:
    amount = Decimal(amount_text)

    if amount < 0:
        print("Amount cannot be negative")
    else:
        print("Amount is valid:", amount)

except InvalidOperation:
    print("Amount is not a valid decimal")
```

Expected output:

```text
Amount is valid: 1250.50
```

The try block contains an operation that may fail. The except block handles the specific decimal conversion error.

## 12.6 Validate the date

```python
from datetime import date

try:
    parsed_date = date.fromisoformat(row["order_date"])
    print("Date is valid:", parsed_date)
except ValueError:
    print("Date must use YYYY-MM-DD")
```

Expected output:

```text
Date is valid: 2026-11-10
```

The exception handler does not make an invalid date valid. It converts a technical failure into a clear validation result.

---

# 13. Validate a complete file row by row

Now combine the ideas. Keep the first implementation simple and visible.

## 13.1 Read all rows into dictionaries

```python
import csv

records = []

with file_path.open("r", encoding="utf-8", newline="") as file:
    reader = csv.DictReader(file)

    for row in reader:
        records.append(row)

print("Records read:", len(records))
```

Expected output:

```text
Records read: 4
```

This example stores all rows in a list. It is appropriate for a small training file. For very large files, process rows incrementally rather than storing everything.

## 13.2 Inspect the first record

```python
print(records[0])
```

Expected output:

```text
{'partner_order_id': 'P-1001', 'customer_id': '501', 'order_date': '2026-11-10', 'status': 'PAID', 'amount': '1250.50'}
```

List indexing begins at zero. Index 0 is the first item.

If the file contains no data rows, `records[0]` raises IndexError. Always check the list length before accessing an item.

## 13.3 Create separate accepted and rejected collections

```python
accepted_records = []
rejected_records = []
```

These lists have different purposes:
- accepted records passed the rules;
- rejected records failed one or more rules.

Do not discard rejected rows. They are important evidence.

## 13.4 Check one record and retain its reason

```python
row = {
    "partner_order_id": "P-1003",
    "customer_id": "503",
    "order_date": "2026-11-10",
    "status": "PAID",
    "amount": "eight hundred"
}

errors = []

try:
    amount = Decimal(row["amount"])
except InvalidOperation:
    errors.append("amount is not a valid decimal")

if len(errors) == 0:
    accepted_records.append(row)
else:
    rejected_records.append({
        "record": row,
        "rejection_reason": "; ".join(errors)
    })

print("Accepted:", len(accepted_records))
print("Rejected:", len(rejected_records))
```

Expected output:

```text
Accepted: 0
Rejected: 1
```

This is a small demonstration. It validates only the amount field. A complete validator must check every required field and business rule.

## 13.5 Why preserve the original row?

The rejected record should retain its original values so that:
- the partner can be told what was wrong;
- an analyst can investigate the issue;
- the record can be corrected and reprocessed;
- the team can prove that it was not silently lost.

A rejection reason should be clear, stable and useful. Avoid vague messages such as `bad row`.

---

# 14. Understand record identity and duplicates

A duplicate is not simply two rows that look similar. We need to know which field or combination of fields identifies one business record.

## 14.1 Example

| partner_order_id | customer_id | amount |
|---|---:|---:|
| P-1001 | 501 | 1250.50 |
| P-1002 | 502 | 800.00 |
| P-1001 | 501 | 1250.50 |

The first and third rows have the same partner order ID.

If the contract says `partner_order_id` is unique within the partner, the third row is a duplicate.

## 14.2 Track keys already seen

```python
seen_order_ids = set()
duplicate_ids = []

for row in records:
    order_id = row["partner_order_id"]

    if order_id in seen_order_ids:
        duplicate_ids.append(order_id)
    else:
        seen_order_ids.add(order_id)

print("Duplicate IDs:", duplicate_ids)
```

A set stores unique values. Checking membership is an efficient way to identify repeated keys.

## 14.3 Duplicate policy is a business decision

Possible policies:
- reject every repeated key;
- keep the first occurrence;
- keep the latest occurrence when a trusted version or update timestamp exists;
- treat repeated delivery of the same file as a file-level duplicate;
- allow repeated keys when the source defines them as separate events.

Never choose a policy only because it is easy to code.

## 14.4 File duplicate and record duplicate are different

**File duplicate:** The same delivery file is received again.

**Record duplicate:** The same business record appears more than once, possibly inside different files.

A pipeline may need both controls:
- file-level delivery tracking;
- record-level unique-key validation.

---

# 15. Write accepted records to a new CSV

Do not overwrite the incoming file. Preserve the original source as evidence.

## 15.1 Define the output fields

```python
output_columns = [
    "partner_order_id",
    "customer_id",
    "order_date",
    "status",
    "amount"
]
```

The output column list defines the output order.

## 15.2 Write the header and records

```python
import csv
from pathlib import Path

accepted_path = Path("accepted/accepted_orders.csv")

with accepted_path.open("w", encoding="utf-8", newline="") as file:
    writer = csv.DictWriter(file, fieldnames=output_columns)
    writer.writeheader()
    writer.writerows(accepted_records)

print("Accepted file created")
```

Explanation:
- `Path(...)` identifies the destination.
- `open("w")` opens for writing and replaces an existing file.
- This is why we must choose the output path carefully.
- `DictWriter` writes dictionaries using the declared field names.
- `writeheader()` writes the column names.
- `writerows(...)` writes multiple dictionary records.
- The `with` block closes the file even if an exception occurs.

**Warning:** Write mode replaces an existing output file. In production, use a unique run-specific temporary file and a controlled publication step.

## 15.3 Write rejected records with reasons

A simple rejected output can include the original fields and a rejection reason.

```python
rejected_path = Path("rejected/rejected_orders.csv")

rejected_columns = output_columns + ["rejection_reason"]

with rejected_path.open("w", encoding="utf-8", newline="") as file:
    writer = csv.DictWriter(file, fieldnames=rejected_columns)
    writer.writeheader()

    for item in rejected_records:
        rejected_row = item["record"].copy()
        rejected_row["rejection_reason"] = item["rejection_reason"]
        writer.writerow(rejected_row)

print("Rejected file created")
```

The `copy()` method creates a shallow copy of the dictionary. We add the rejection reason to the copy, not to the original record.

## 15.4 Inspect the output

Open the accepted and rejected files in VS Code or a spreadsheet application.

Check:
- header is present;
- values are in the correct columns;
- accepted file contains only accepted rows;
- rejected file contains a reason;
- original input file has not been overwritten.

Do not use a spreadsheet application to save large source files unless the workflow explicitly requires it. Spreadsheet applications can change formatting, leading zeros and date representations.

---

# 16. Reconcile the ingestion result

Reconciliation means comparing counts or values to establish whether processing accounted for the input.

## 16.1 The basic record-count equation

`Input records = Accepted records + Rejected records`

For example:
- input = 4;
- accepted = 3;
- rejected = 1.

Then:

`4 = 3 + 1`

This proves that the in-memory classification accounted for all four records. It does not by itself prove that the output files were written correctly.

## 16.2 Calculate counts

```python
input_count = len(records)
accepted_count = len(accepted_records)
rejected_count = len(rejected_records)

print("Input:", input_count)
print("Accepted:", accepted_count)
print("Rejected:", rejected_count)
print("Accounted for:", accepted_count + rejected_count)
```

Expected output depends on the records and validation rules used in the current notebook.

## 16.3 Check the equation

```python
if input_count == accepted_count + rejected_count:
    print("Record reconciliation passed")
else:
    print("Record reconciliation failed")
```

A failed reconciliation means some records were not classified or the counts were measured at different stages.

## 16.4 Count records after writing

Reopen the accepted output and count its data rows. Do not simply trust the number of rows that the program intended to write.

```python
with accepted_path.open("r", encoding="utf-8", newline="") as file:
    accepted_reader = csv.DictReader(file)
    accepted_written_count = sum(1 for row in accepted_reader)

print("Accepted rows written:", accepted_written_count)
```

This counts rows in the saved file, excluding the header because DictReader treats the first row as field names.

## 16.5 Compare intended and actual output

```python
if accepted_written_count == accepted_count:
    print("Accepted output count matches")
else:
    print("Accepted output count does not match")
```

This checks output row count. A row count can still match while values are wrong, so production pipelines should also compare keys, totals, schemas and relevant checksums.

---

# 17. Read JSON files safely

CSV is not the only file format. Now read a JSON delivery.

## 17.1 Create a JSON array file

Create `input/delivery_updates.json`:

```json
[
  {
    "tracking_id": "T-9001",
    "order_id": "P-1001",
    "delivery_status": "OUT_FOR_DELIVERY"
  },
  {
    "tracking_id": "T-9002",
    "order_id": "P-1002",
    "delivery_status": "DELIVERED"
  }
]
```

## 17.2 Import json and open the file

```python
import json
from pathlib import Path

json_path = Path("input/delivery_updates.json")

with json_path.open("r", encoding="utf-8") as file:
    delivery_data = json.load(file)

print(type(delivery_data))
print(len(delivery_data))
```

Expected output:

```text
<class 'list'>
2
```

Explanation:
- `json` is a standard Python module.
- `json.load(file)` parses JSON from an open file.
- The outer JSON array becomes a Python list.
- The list contains two dictionaries.

## 17.3 Read values from one object

```python
first_delivery = delivery_data[0]

print(first_delivery["tracking_id"])
print(first_delivery["delivery_status"])
```

Expected output:

```text
T-9001
OUT_FOR_DELIVERY
```

The first object is at index zero. Dictionary keys retrieve individual fields.

## 17.4 Handle malformed JSON

If a JSON file has a missing comma, invalid quote or incomplete structure, parsing raises a JSON decoding error.

```python
try:
    with json_path.open("r", encoding="utf-8") as file:
        delivery_data = json.load(file)

    print("JSON file parsed")

except json.JSONDecodeError as error:
    print("JSON file is malformed")
    print(error)
```

Do not treat malformed JSON as an empty successful delivery. Record the filename, error and processing status.

## 17.5 Validate the JSON root type

```python
if isinstance(delivery_data, list):
    print("Root is an array")
elif isinstance(delivery_data, dict):
    print("Root is an object")
else:
    print("Unexpected JSON root type")
```

A valid JSON document may still have the wrong structure for the consumer. Parsing success and contract validation are separate checks.

## 17.6 Validate required JSON keys

```python
required_json_keys = ["tracking_id", "order_id", "delivery_status"]

for record in delivery_data:
    missing_keys = []

    for key in required_json_keys:
        if key not in record:
            missing_keys.append(key)

    print(record.get("tracking_id"), missing_keys)
```

This checks whether required keys exist. A complete validator should also check whether their values are empty and whether the values follow the business contract.

---

# 18. Process multiple incoming files

A partner may send one file per region or one file per business hour.

## 18.1 Discover multiple CSV files

Place two CSV files in the input folder, each following the same contract.

```python
csv_files = sorted(Path("input").glob("partner_orders_*.csv"))

print("Files discovered:", len(csv_files))

for file_path in csv_files:
    print(file_path.name)
```

The output depends on the files currently in the folder.

## 18.2 Why should each file be identified separately?

If one file fails, we need to know which delivery failed.

Record at least:
- source filename;
- file size;
- discovered timestamp;
- processing start and end;
- input row count;
- accepted count;
- rejected count;
- processing status;
- error summary.

A single combined count for many files can hide one missing or duplicated delivery.

## 18.3 Read each file

```python
all_records = []

for file_path in csv_files:
    with file_path.open("r", encoding="utf-8", newline="") as file:
        reader = csv.DictReader(file)

        for row in reader:
            row["source_file"] = file_path.name
            all_records.append(row)

print("Total records:", len(all_records))
```

This example adds source filename as metadata. It is useful when several files are combined.

Do not add a field to the business output unless the destination schema supports it. Source metadata may belong in a separate audit table or output column.

## 18.4 Avoid mixing incompatible file contracts

Suppose one file contains:
- `order_id`;
- `amount`.

Another contains:
- `tracking_id`;
- `delivery_status`.

Both are CSV files, but they represent different datasets. Do not combine them merely because the extensions match.

Group files by feed type and contract.

---

# 19. Handle empty, partial and malformed deliveries

These cases must be distinguished.

## 19.1 Missing file

The expected file does not exist.

Possible cause:
- partner did not deliver;
- wrong filename;
- upload failed;
- file arrived in a different folder.

Action: record a missing-delivery status and follow the agreed operational process.

## 19.2 Zero-byte file

The file exists but contains no bytes.

Possible cause:
- interrupted upload;
- producer generated an empty file;
- transfer issue.

Action: do not parse it as a successful delivery without an explicit empty-file contract.

## 19.3 Header-only file

The file has a header but no data records.

This may be a valid empty delivery if the producer is allowed to send no records.

Action: distinguish a valid zero-record delivery from an unexpected missing delivery.

## 19.4 Partial file

A file may be visible before upload finishes.

A consumer that reads it too early may see incomplete content.

Safer delivery patterns include:
- producer writes to a temporary extension and renames after completion;
- producer writes a completion marker;
- transfer process publishes only completed files;
- consumer checks stable size and delivery metadata.

A file-size check alone is not a perfect completion guarantee.

## 19.5 Malformed file

Examples:
- invalid encoding;
- broken quoting;
- missing header;
- inconsistent columns;
- invalid JSON syntax.

Action: quarantine the file, retain evidence, and report the exact failure. Do not silently skip the file.

---

# 20. Avoid processing the same file twice

A scheduled pipeline may run again after a failure. The same input file can still be present.

## 20.1 Why duplicate processing happens

- the job was retried;
- the scheduler started the same run twice;
- the previous run failed after writing output;
- the partner resent the same file;
- an operator manually restarted processing.

## 20.2 Track processed files

For a learning demonstration, maintain a set of filenames:

```python
processed_files = {"partner_orders_2026-11-09.csv"}

incoming_name = "partner_orders_2026-11-10.csv"

if incoming_name in processed_files:
    print("File was already processed")
else:
    print("File is new")
```

Expected output:

```text
File is new
```

A Python set exists only while the process is running unless we save it. Production systems need durable processing metadata.

## 20.3 Why filename alone may be insufficient

A partner could send a corrected file using the same filename. Or the same content could arrive under a new filename.

A robust file identity may include:
- partner identifier;
- delivery identifier;
- business date;
- file name;
- content checksum;
- version number.

The exact identity rule must be agreed with the source owner.

## 20.4 Understand a checksum

A checksum is a calculated value based on file bytes. It can help detect whether two files have identical content.

A cryptographic hash such as SHA-256 is commonly used.

```python
import hashlib

file_hash = hashlib.sha256()

with file_path.open("rb") as file:
    for chunk in iter(lambda: file.read(1024 * 1024), b""):
        file_hash.update(chunk)

print(file_hash.hexdigest())
```

Explanation:
- `hashlib` provides hash algorithms.
- `sha256()` creates a SHA-256 hash object.
- `"rb"` opens the file in binary read mode.
- Reading in chunks avoids loading a very large file into memory.
- `update(chunk)` adds bytes to the calculation.
- `hexdigest()` returns the digest as readable hexadecimal text.

The hash changes when file bytes change. It does not prove that the business data is correct.

## 20.5 Store the file processing result durably

A production file registry might store:
- delivery ID;
- source partner;
- filename;
- checksum;
- received time;
- processing status;
- row counts;
- output location;
- error details.

This registry helps make reruns controlled and auditable.

---

# 21. Write files safely

Writing output is a separate engineering responsibility.

## 21.1 Why direct final-file writing can be risky

Imagine the process writes half of the output and then crashes. A downstream reader might see a partial file with the final expected name.

Safer pattern:
1. write to a temporary filename;
2. close the file;
3. validate the temporary output;
4. rename or publish it as the final output;
5. record success only after publication.

## 21.2 Write a temporary file

```python
temporary_path = Path("accepted/accepted_orders.tmp")
final_path = Path("accepted/accepted_orders.csv")

with temporary_path.open("w", encoding="utf-8", newline="") as file:
    writer = csv.DictWriter(file, fieldnames=output_columns)
    writer.writeheader()
    writer.writerows(accepted_records)

print(temporary_path.exists())
```

Expected output:

```text
True
```

## 21.3 Publish after validation

```python
if temporary_path.exists():
    temporary_path.replace(final_path)
    print("Output published")
```

The rename/replace operation is typically atomic when source and destination are on the same filesystem, but exact guarantees depend on the filesystem and environment. Do not assume the same behavior across network storage systems.

## 21.4 Avoid accidental overwrite

Before replacing an existing output, define whether the run:
- should create a new version;
- should replace the prior output;
- should fail if the output exists;
- should append to a managed dataset.

Appending blindly can duplicate records on rerun.

---

# 22. Understand large-file processing

A small CSV can be read into a list. A multi-gigabyte file needs a different approach.

## 22.1 Load everything into memory

```python
with file_path.open("r", encoding="utf-8", newline="") as file:
    all_text = file.read()
```

This loads the complete file content into memory.

If a file is 5 GB, the process needs enough available memory for the text and the objects created from it. Actual memory usage can be significantly larger than file size.

## 22.2 Process one row at a time

```python
with file_path.open("r", encoding="utf-8", newline="") as file:
    reader = csv.DictReader(file)

    for row in reader:
        print(row["partner_order_id"])
```

The reader yields one record at a time. This is more memory-efficient than storing every row in a list.

## 22.3 Write each accepted row as it is read

```python
with file_path.open("r", encoding="utf-8", newline="") as source_file:
    reader = csv.DictReader(source_file)

    with accepted_path.open("w", encoding="utf-8", newline="") as output_file:
        writer = csv.DictWriter(output_file, fieldnames=output_columns)
        writer.writeheader()

        for row in reader:
            if row["status"] in ["PAID", "SHIPPED", "PENDING", "CANCELLED"]:
                writer.writerow(row)
```

This demonstrates streaming-style file processing: read a record, make a decision, write it, then move to the next record.

This is not yet a complete production validator. It checks only status membership and does not preserve rejected rows.

## 22.4 Memory trade-off

| Approach | Memory use | Simplicity | Suitable for |
|---|---|---|---|
| `read()` | High for large files | Very simple | Tiny text examples |
| `readlines()` | High | Simple | Small files |
| List of dictionaries | Grows with record count | Easy to inspect | Small/medium datasets |
| Iterate CSV reader | Lower and bounded | Good | Large files |
| Chunked DataFrame reading | Bounded by chunk size | Requires Pandas | Tabular transformation workloads |

Choose based on file size, record width, transformation complexity and available memory.

---

# 23. Use Pandas when tabular analysis is useful

Pandas is an external Python package. It is useful for working with rows and columns as DataFrames.

## 23.1 Install Pandas

Run in CMD or the notebook terminal:

```bat
python -m pip install pandas
```

If the notebook uses a different Python environment, install into that same environment.

## 23.2 Read CSV

```python
import pandas as pd

data = pd.read_csv("input/partner_orders_2026-11-10.csv")

print(data.head())
```

Expected display contains the first five records or fewer if the file has fewer records. Pandas may display a formatted table in Jupyter.

## 23.3 Understand a DataFrame

A DataFrame is a two-dimensional labeled table:
- rows represent records;
- columns represent fields;
- each column has a name;
- each column has a data type.

## 23.4 Inspect shape

```python
print(data.shape)
```

For the four-record, five-column sample, expected output is:

```text
(4, 5)
```

The first number is the row count. The second is the column count.

## 23.5 Inspect column names

```python
print(data.columns.tolist())
```

Expected output:

```text
['partner_order_id', 'customer_id', 'order_date', 'status', 'amount']
```

## 23.6 Inspect data types

```python
print(data.dtypes)
```

The exact inferred types depend on the data. Pandas may infer integer or floating-point types, while identifiers and dates may need explicit treatment.

Never rely blindly on automatic inference for important business fields.

## 23.7 Read all fields as text first

```python
data = pd.read_csv(
    "input/partner_orders_2026-11-10.csv",
    dtype=str
)

print(data.dtypes)
```

This keeps source values as text for controlled conversion. It can be useful when leading zeros or inconsistent values matter.

## 23.8 Read a large CSV in chunks

```python
for chunk in pd.read_csv(
    "input/partner_orders_2026-11-10.csv",
    chunksize=2
):
    print(chunk)
```

The `chunksize` parameter sets the number of rows per chunk in this example. A real chunk size should be selected through measurement and workload constraints.

---

# 24. Common errors and how to fix them

## 24.1 FileNotFoundError

**Meaning:** Python could not find the path.

Check:
- current directory;
- spelling;
- extension;
- folder location;
- whether the delivery arrived.

Do not solve this by creating an empty file unless an empty file is genuinely the expected source.

## 24.2 PermissionError

**Meaning:** The process does not have permission to read or write the path.

Possible causes:
- file is locked by another application;
- user lacks folder permission;
- destination is read-only;
- another process owns the file.

Fix permissions or coordinate with the process that owns the file. Do not run everything as administrator by default.

## 24.3 UnicodeDecodeError

**Meaning:** Bytes could not be decoded using the selected encoding.

Actions:
1. Confirm the partner's encoding contract.
2. Inspect file metadata or request a sample.
3. Use the agreed encoding.
4. Record decoding failures.
5. Avoid lossy replacement unless the business explicitly approves it.

## 24.4 csv.Error

**Meaning:** The CSV parser encountered a parsing issue.

Check:
- quote characters;
- escape conventions;
- delimiter;
- malformed rows;
- producer's CSV export settings.

## 24.5 KeyError

**Meaning:** Code requested a dictionary key that does not exist.

Example:
`row["customer_id"]`

Possible cause: header was renamed to `buyer_id`.

Fix:
- validate the header first;
- use only approved mappings;
- do not hide the problem by assigning a random default.

## 24.6 ValueError

**Meaning:** A conversion or operation received an invalid value.

Example:
`int("five")`

Fix:
- retain the original text;
- validate before conversion;
- record a useful reason;
- do not silently convert invalid values to zero.

## 24.7 JSONDecodeError

**Meaning:** The JSON text is malformed.

Check:
- missing comma;
- invalid quotes;
- trailing content;
- truncated upload;
- incorrect file format.

## 24.8 Empty output file

Possible causes:
- input had only a header;
- filter excluded all rows;
- wrong input file was selected;
- output was written before data was appended;
- the process failed before writing.

Check input count, filter logic, accepted count and output count.

---

# 25. Design rejection reasons that help people fix data

A rejected record should carry a reason that is understandable and actionable.

Weak reason:
`Invalid row`

Better reasons:
- `partner_order_id is missing`;
- `amount is not a valid decimal`;
- `order_date must use YYYY-MM-DD`;
- `status is not in the approved status list`;
- `partner_order_id is duplicated in this delivery`.

A record can have more than one problem. Preserve all applicable reasons when practical.

## 25.1 One error or multiple errors?

**Stop at the first error** is simpler. It can be appropriate when later checks depend on the earlier field.

**Collect multiple errors** gives the partner a more complete correction report. It requires careful validation so one failed conversion does not cause unrelated checks to crash.

## 25.2 Never invent corrected values

If amount is `eight hundred`, do not silently replace it with 800.

If customer ID is missing, do not guess the customer from the name.

If status is unknown, do not convert it to PAID.

A data engineer implements approved rules; they do not manufacture business facts.

---

# 26. File security and privacy basics

Files can contain personal, financial or confidential information.

Good practices:
- restrict access to input, accepted and rejected folders;
- do not upload real customer data to public repositories;
- use synthetic data for learning;
- do not print sensitive values in logs;
- protect files in transit and at rest according to company policy;
- define retention and deletion rules;
- validate filenames and paths before opening files;
- do not trust file contents merely because the sender is known.

A filename is untrusted input. Avoid allowing external filenames to control arbitrary filesystem paths.

---

# 27. Operational metadata for every delivery

A file pipeline should produce a record of what happened.

Useful metadata:

| Field | Purpose |
|---|---|
| run_id | Identifies one execution |
| source_name | Identifies the partner/feed |
| file_name | Identifies the received file |
| file_size_bytes | Helps investigate delivery changes |
| checksum | Helps identify identical content |
| received_at | When the file arrived |
| started_at | When processing began |
| completed_at | When processing ended |
| input_count | Number of parsed data records |
| accepted_count | Number of valid records |
| rejected_count | Number of invalid records |
| output_path | Where results were written |
| status | SUCCESS, PARTIAL or FAILED |
| error_summary | Short failure description |

This metadata is separate from business records. It allows an operator to understand a run without opening every row.

---

# 28. Practical learning checks

Complete these checks using the sample files from this lesson. They are part of the class learning, not the separate project workbook.

## Check 1 — Identify the file structure

Answer:
1. Which row contains the CSV header?
2. How many data records are in the sample?
3. Which field identifies the partner order?
4. Which field represents money?
5. What encoding did we choose?

Expected reasoning:
- Header is the first row.
- There are four sample data records.
- `partner_order_id` is the record key under this contract.
- `amount` represents money.
- UTF-8 is the agreed encoding for this exercise.

## Check 2 — Predict the Python type

For a CSV row read by DictReader, what is the type of `row["amount"]`?

Expected answer: string.

Why? CSV is parsed as text. We must convert it to Decimal when money calculations are needed.

## Check 3 — Identify a missing field

Given:

```csv
partner_order_id,customer_id,amount
P-2001,,100.00
```

Which field is empty?

Expected answer: customer_id.

Is the record accepted? Under the contract in this lesson, no. customer_id is required.

## Check 4 — Identify malformed amount

Given:

```csv
partner_order_id,customer_id,amount
P-2002,502,one hundred
```

What happens when we convert amount to Decimal?

Expected answer: Decimal conversion raises InvalidOperation. Record the error and reject the record under the current rules.

## Check 5 — Explain the count equation

If 250 records are read, 240 are accepted and 10 are rejected, does the record-count reconciliation pass?

Expected answer: yes, because 250 = 240 + 10.

What does this not prove? It does not prove every accepted field is correct or that the output file contains the intended values.

## Check 6 — Identify a duplicate

Two rows have the same partner_order_id. What should the pipeline do?

Expected answer: follow the documented duplicate policy. Do not assume that the first or last row is always correct.

## Check 7 — Choose a reader

Which is more readable for a named business field: `row[4]` or `row["amount"]`?

Expected answer: `row["amount"]`, because the field name communicates meaning.

## Check 8 — Explain the difference between parse and validate

Parsing converts file text into Python structures. Validation checks whether those structures follow the contract.

A JSON file can parse successfully and still be invalid because required keys are missing.

## Check 9 — Explain why we archive the original

The original file provides source evidence. If a transformation or validation rule is later questioned, the team can inspect what the partner actually sent.

## Check 10 — Explain why a file should not be marked processed too early

If processing is marked complete before output is safely written and verified, a crash may cause the system to skip a delivery whose data was never published.

---

# 29. Guided practice variations

Use copies of the sample file. Keep the original clean sample unchanged.

## Variation A — Change the delimiter

1. Create a copy of the file.
2. Replace commas with semicolons.
3. Read it using the default CSV reader.
4. Inspect the parsed header.
5. Change the delimiter parameter to a semicolon.
6. Compare the results.

Learning point: the parser must match the file contract.

## Variation B — Remove a required column

1. Remove `customer_id` from the header.
2. Keep the remaining data.
3. Run the header validation.
4. Inspect the missing-column result.
5. Decide whether the file should be rejected as a whole.

Learning point: schema validation should happen before record processing when a required column is absent.

## Variation C — Add an invalid amount

1. Change one amount to `unknown`.
2. Read the row.
3. Attempt Decimal conversion.
4. Catch the conversion exception.
5. Record the row and a clear rejection reason.

Learning point: preserve the raw value and explain the failure.

## Variation D — Add a duplicate key

1. Copy one row.
2. Keep the same partner_order_id.
3. Run the duplicate-key check.
4. Inspect which record is identified as repeated.
5. Apply only the duplicate policy defined by the contract.

Learning point: uniqueness is a business rule, not a property automatically guaranteed by CSV.

## Variation E — Create a header-only file

1. Create a new CSV file.
2. Add only the header.
3. Read it with DictReader.
4. Count the records.
5. Explain why zero records can be valid or invalid depending on the delivery agreement.

Learning point: distinguish an empty business delivery from a missing or failed delivery.

## Variation F — Create malformed JSON

1. Copy the JSON sample.
2. Remove a comma between two properties.
3. Attempt to load it.
4. Catch JSONDecodeError.
5. Record the filename and error.

Learning point: syntactically valid JSON and a valid business contract are separate checks.

---

# 30. Student self-check before finishing

You should be able to explain these without copying the definitions.

- What is file-based ingestion?
- What is a file contract?
- What is the difference between a file and a record?
- What is the purpose of a CSV header?
- Why does CSV not guarantee data types?
- What is a delimiter?
- Why do quoted commas matter?
- What is the difference between csv.reader and csv.DictReader?
- What does pathlib.Path represent?
- What is a relative path?
- What does FileNotFoundError mean?
- What is the difference between an empty file and a header-only file?
- What is JSON?
- What is the difference between a JSON object and an array?
- What does json.load do?
- What is schema validation?
- What is record validation?
- Why should we use Decimal for money?
- Why should identifiers sometimes remain strings?
- What is a duplicate business key?
- Why do we preserve rejected rows?
- What does reconciliation prove?
- Why is a row count not enough to prove data correctness?
- Why should large files be processed incrementally?
- Why should original source files be retained?
- Why should a pipeline track file identity?
- Why is a checksum useful?
- Why should output be published only after it is complete?
- Why must business rules be agreed rather than guessed?

If you cannot explain one item, return to the section and rerun its example.

---

# 31. Key takeaways

1. A file arriving is only the beginning of ingestion.
2. The file contract defines the expected format, schema and business rules.
3. CSV is text; values require controlled type conversion.
4. JSON can represent objects, arrays and nested structures.
5. Use standard parsers rather than manually splitting text.
6. Validate file structure before validating individual records.
7. Preserve rejected records and clear rejection reasons.
8. Define duplicate behavior with the source owner.
9. Write accepted and rejected outputs separately.
10. Reconcile input, accepted and rejected counts.
11. Count checks are necessary but do not prove every value is correct.
12. Process large files in a memory-aware way.
13. Track each delivery so reruns are controlled.
14. Preserve original files and publish outputs safely.
15. Protect sensitive data and retain only what policy permits.

## What comes next

Day 32 teaches API-based data ingestion. The source changes from a partner file to an HTTP API, but the engineering questions remain familiar: identify the source, understand its contract, retrieve data safely, validate it, handle failures, and prove completeness.

**End of Day 31 class notes.**
