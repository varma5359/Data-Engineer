# Day 9: Data Formats and File Processing

**Phase:** 2 — Python for Data Engineering  
**Mini-project:** Multi-Format Data Reader  
**Learning time:** 2–3 hours

## 1. Learning outcomes

By the end of this lesson, you can:

- Explain why data formats exist and how serialization works.
- Read and write CSV using Python's `csv` module.
- Read and write JSON using Python's `json` module.
- Understand XML elements, attributes and nested records.
- Explain Parquet, columnar storage and its typical use cases.
- Compare formats based on structure, size, readability and analytics workload.
- Check basic schema expectations after loading a file.

## 2. Why data formats matter

A data engineer rarely receives every source in the same format.

| Source system | Possible delivery format |
|---|---|
| Finance team export | CSV |
| REST API | JSON |
| Older enterprise application | XML |
| Data lake / analytics platform | Parquet |

A file extension tells you the intended format, but it does not prove that the file is valid. A file named `orders.csv` may contain malformed rows, missing columns or incorrect values.

**Pipeline view:** Source file -> Parse -> Validate structure -> Convert types -> Transform -> Store or deliver.

## 3. Serialization and deserialization

- **Serialization:** Convert an in-memory object into a format that can be stored or transmitted.
- **Deserialization:** Read the stored/transmitted representation and reconstruct an in-memory object.

Example:

```text
Python dictionary -> JSON text -> file
file -> JSON parser -> Python dictionary
```

The parser reconstructs data structure; it does not automatically guarantee business correctness.

## 4. Prepare sample order data

We will reuse the same small retail scenario in each format.

```python
orders = [
    {"order_id": 501, "customer": "Ravi", "amount": 1200.50, "status": "PAID"},
    {"order_id": 502, "customer": "Anu", "amount": 800.00, "status": "PENDING"},
    {"order_id": 503, "customer": "Kiran", "amount": 450.00, "status": "PAID"}
]

print("Number of orders:", len(orders))
print(orders[0])
```

**Expected output**

```text
Number of orders: 3
{'order_id': 501, 'customer': 'Ravi', 'amount': 1200.5, 'status': 'PAID'}
```

**Explanation:** A list holds multiple records. Each dictionary represents one order. The same logical records will be saved in different file formats.

## 5. CSV: Comma-Separated Values

### 5.1 What is CSV?

CSV stores tabular records as rows and fields as columns. It is widely supported by spreadsheets, databases and data tools.

CSV is simple, but it does not reliably preserve data types. When reading with Python's standard CSV reader, values are returned as strings.

### 5.2 Write a CSV file

```python
import csv

csv_file = "orders.csv"
field_names = ["order_id", "customer", "amount", "status"]

with open(csv_file, "w", newline="", encoding="utf-8") as file:
    writer = csv.DictWriter(file, fieldnames=field_names)
    writer.writeheader()
    writer.writerows(orders)

print("CSV file created")
```

**Expected output**

```text
CSV file created
```

**Line-by-line explanation**

- `import csv` loads Python's standard CSV support.
- `newline=""` is the recommended way to open CSV files for writing.
- `DictWriter` maps dictionary keys to CSV columns.
- `fieldnames` defines the column order.
- `writeheader()` writes the header row.
- `writerows()` writes all dictionaries as records.

### 5.3 Inspect the actual CSV text

```python
with open(csv_file, "r", encoding="utf-8") as file:
    print(file.read())
```

**Expected output**

```text
order_id,customer,amount,status
501,Ravi,1200.5,PAID
502,Anu,800.0,PENDING
503,Kiran,450.0,PAID
```

### 5.4 Read CSV records

```python
with open(csv_file, "r", newline="", encoding="utf-8") as file:
    reader = csv.DictReader(file)
    csv_orders = list(reader)

print(csv_orders[0])
print(type(csv_orders[0]["order_id"]))
print(type(csv_orders[0]["amount"]))
```

**Expected output**

```text
{'order_id': '501', 'customer': 'Ravi', 'amount': '1200.5', 'status': 'PAID'}
<class 'str'>
<class 'str'>
```

**Important interpretation:** CSV does not tell Python that `501` should be an integer or `1200.5` should be a float. Explicitly convert and validate values before calculations.

### 5.5 Convert CSV values

```python
for row in csv_orders:
    row["order_id"] = int(row["order_id"])
    row["amount"] = float(row["amount"])

print(csv_orders[0])
print(type(csv_orders[0]["order_id"]))
print(type(csv_orders[0]["amount"]))
```

**Expected output**

```text
{'order_id': 501, 'customer': 'Ravi', 'amount': 1200.5, 'status': 'PAID'}
<class 'int'>
<class 'float'>
```

**Common issue:** An empty amount such as `""` cannot be converted directly using `float()`. Check missing values and define a rejection/default policy before conversion.

### 5.6 CSV edge cases

- Commas inside a value: `"Hyderabad, Telangana"`.
- Quotes inside a text field.
- Empty fields.
- Different delimiters such as semicolon.
- Duplicate or missing column names.
- Leading zeros in identifiers such as `"00051"`.
- Different text encodings.

Do not parse CSV using `line.split(",")`. A quoted field can contain a comma, so use a CSV parser.

## 6. JSON: JavaScript Object Notation

### 6.1 What is JSON?

JSON represents objects, arrays, strings, numbers, booleans and null values. It is common in REST APIs and configuration files.

Python mapping:

| JSON | Python |
|---|---|
| object | dict |
| array | list |
| string | str |
| number | int / float |
| true / false | True / False |
| null | None |

### 6.2 Convert a Python object to JSON text

```python
import json

json_text = json.dumps(orders, indent=2)
print(json_text)
```

**Expected output**

A readable JSON array containing the three order objects, one object per order. Each object contains `order_id`, `customer`, `amount` and `status`.

**Explanation**

- `dumps()` converts a Python object to a JSON string.
- `indent=2` formats the text for readability.
- The original Python list remains available in `orders`.

### 6.3 Write JSON to a file

```python
json_file = "orders.json"

with open(json_file, "w", encoding="utf-8") as file:
    json.dump(orders, file, indent=2)

print("JSON file created")
```

**Expected output**

```text
JSON file created
```

Difference:
- `json.dumps()` returns a JSON string.
- `json.dump()` writes JSON directly to a file object.

### 6.4 Read JSON from a file

```python
with open(json_file, "r", encoding="utf-8") as file:
    json_orders = json.load(file)

print(type(json_orders))
print(len(json_orders))
print(json_orders[0])
```

**Expected output**

```text
<class 'list'>
3
{'order_id': 501, 'customer': 'Ravi', 'amount': 1200.5, 'status': 'PAID'}
```

**Interpretation:** Unlike the basic CSV reader example, JSON preserves numeric values as numbers when they were written as numbers.

### 6.5 Read a JSON string

```python
sample_json = '{"order_id": 504, "amount": 250, "status": "PAID"}'
parsed_order = json.loads(sample_json)

print(parsed_order["order_id"])
print(type(parsed_order["amount"]))
```

**Expected output**

```text
504
<class 'int'>
```

`loads()` parses a JSON string; `load()` parses JSON from a file object.

### 6.6 JSON validation considerations

- The top-level value may be an object or an array.
- A key may be missing.
- A value may be null.
- A field may have the wrong type.
- Nested objects may not have the expected structure.
- Invalid JSON raises `json.JSONDecodeError`.

Example safe parsing:

```python
invalid_json = '{"order_id": 501,}'

try:
    parsed_data = json.loads(invalid_json)
    print(parsed_data)
except json.JSONDecodeError as error:
    print("Invalid JSON:", error.msg)
```

**Expected output**

```text
Invalid JSON: Expecting property name enclosed in double quotes
```

The exact position information is available on the exception object. Do not silently replace invalid source data with an empty list; record and investigate the parsing failure.

## 7. XML: Extensible Markup Language

### 7.1 What is XML?

XML stores structured data using nested tags. It is common in integrations with older enterprise systems and some document-oriented exchanges.

Example structure:

```xml
<orders>
    <order id="501">
        <customer>Ravi</customer>
        <amount>1200.50</amount>
        <status>PAID</status>
    </order>
</orders>
```

- `orders` is the root element.
- `order` is a child element.
- `id` is an attribute.
- `customer`, `amount` and `status` are child elements.

### 7.2 Create an XML file

```python
from xml.etree.ElementTree import Element, SubElement, ElementTree

root = Element("orders")

for order in orders:
    order_element = SubElement(root, "order", id=str(order["order_id"]))
    SubElement(order_element, "customer").text = order["customer"]
    SubElement(order_element, "amount").text = str(order["amount"])
    SubElement(order_element, "status").text = order["status"]

xml_file = "orders.xml"
ElementTree(root).write(xml_file, encoding="unicode", xml_declaration=True)

print("XML file created")
```

**Expected output**

```text
XML file created
```

**Explanation**

- `Element("orders")` creates the root.
- `SubElement()` adds a nested element.
- The `id` value is an attribute.
- Element text is stored as text, so numeric values must be converted back when required.
- `ElementTree.write()` saves the tree.

### 7.3 Read XML records

```python
import xml.etree.ElementTree as ET

tree = ET.parse(xml_file)
xml_root = tree.getroot()

print(xml_root.tag)
print(len(xml_root))
```

**Expected output**

```text
orders
3
```

### 7.4 Extract fields and convert types

```python
xml_orders = []

for order_element in xml_root.findall("order"):
    order_record = {
        "order_id": int(order_element.get("id")),
        "customer": order_element.findtext("customer"),
        "amount": float(order_element.findtext("amount")),
        "status": order_element.findtext("status")
    }
    xml_orders.append(order_record)

print(xml_orders[0])
```

**Expected output**

```text
{'order_id': 501, 'customer': 'Ravi', 'amount': 1200.5, 'status': 'PAID'}
```

**Interpretation:** XML element text is generally read as text. Convert values only after checking that the element exists and contains a valid value.

### 7.5 XML mistakes

- Looking for the wrong tag name.
- Assuming every optional element exists.
- Confusing an attribute with a child element.
- Forgetting to convert text to numeric or date types.
- Assuming all XML documents have the same namespace or nesting.

For untrusted XML from external sources, use a parser configuration and library appropriate to the security requirements. Do not parse arbitrary untrusted XML without considering entity-expansion and external-resource risks.

## 8. Parquet: columnar storage

### 8.1 What is Parquet?

Parquet is a columnar file format designed for analytical workloads. Instead of storing values row by row like a typical CSV representation, it organizes data by columns and stores schema/type information.

Why this matters:

- Analytics often reads only a few columns from a wide table.
- Columnar storage can reduce unnecessary I/O.
- Compression and encoding can reduce storage size.
- Data types and nested structures are represented more explicitly than in plain CSV.

### 8.2 Install the Parquet engine

Pandas commonly uses PyArrow or Fastparquet to read/write Parquet.

```bash
python -m pip install pandas pyarrow
```

If using Jupyter, install into the same environment as the active kernel. Restart the kernel if the package was installed while the notebook was open.

### 8.3 Create a DataFrame and write Parquet

```python
import pandas as pd

orders_df = pd.DataFrame(orders)
parquet_file = "orders.parquet"

orders_df.to_parquet(parquet_file, index=False, engine="pyarrow")

print("Parquet file created")
```

**Expected output**

```text
Parquet file created
```

### 8.4 Read Parquet

```python
parquet_orders = pd.read_parquet(parquet_file, engine="pyarrow")

print(parquet_orders)
print(parquet_orders.dtypes)
```

**Expected output**

A three-row DataFrame with the order columns. The numeric fields are represented with numeric dtypes rather than plain text strings.

**Explanation**

- `to_parquet()` writes a DataFrame in Parquet format.
- `index=False` avoids storing the DataFrame's row index as an extra column.
- `read_parquet()` reads the file into a DataFrame.
- The Parquet engine must be installed in the active environment.

### 8.5 Read selected columns

```python
amounts_only = pd.read_parquet(
    parquet_file,
    columns=["order_id", "amount"],
    engine="pyarrow"
)

print(amounts_only)
```

**Expected output**

```text
   order_id  amount
0       501  1200.5
1       502   800.0
2       503   450.0
```

**Interpretation:** Selecting only needed columns is a common analytical pattern. The Parquet format and reader can avoid loading unrelated columns.

## 9. Compare CSV, JSON, XML and Parquet

| Feature | CSV | JSON | XML | Parquet |
|---|---|---|---|---|
| Main structure | Rows and columns | Objects and arrays | Nested elements | Columnar table |
| Human readability | High | High | High but verbose | Low without a reader |
| Type preservation | Limited | Basic JSON types | Mostly text/attributes | Strong schema/type support |
| Nested data | Awkward | Natural | Natural | Supported |
| Common use | Simple data exchange | APIs and documents | Enterprise integrations | Analytics and data lakes |
| Typical analytics efficiency | Depends on workload | Usually not ideal for large tabular scans | Usually not ideal for large tabular scans | Often efficient for column-based analytics |

There is no universally best format. Choose based on the producer, consumer, schema, size, query pattern and interoperability requirements.

## 10. Mini-project: Multi-Format Data Reader

### 10.1 Business requirement

A data engineer receives order data from different source systems. The goal is to load the supported formats into a common Python representation and compare record counts.

### 10.2 Read CSV into dictionaries

```python
with open(csv_file, "r", newline="", encoding="utf-8") as file:
    csv_data = list(csv.DictReader(file))

print("CSV records:", len(csv_data))
```

**Expected output**

```text
CSV records: 3
```

### 10.3 Read JSON

```python
with open(json_file, "r", encoding="utf-8") as file:
    json_data = json.load(file)

print("JSON records:", len(json_data))
```

**Expected output**

```text
JSON records: 3
```

### 10.4 Read XML

```python
xml_tree = ET.parse(xml_file)
xml_data = xml_tree.getroot().findall("order")

print("XML records:", len(xml_data))
```

**Expected output**

```text
XML records: 3
```

### 10.5 Read Parquet

```python
parquet_data = pd.read_parquet(parquet_file, engine="pyarrow")

print("Parquet records:", len(parquet_data))
```

**Expected output**

```text
Parquet records: 3
```

### 10.6 Compare the record counts

```python
format_counts = {
    "CSV": len(csv_data),
    "JSON": len(json_data),
    "XML": len(xml_data),
    "Parquet": len(parquet_data)
}

for format_name, record_count in format_counts.items():
    print(format_name, "->", record_count)
```

**Expected output**

```text
CSV -> 3
JSON -> 3
XML -> 3
Parquet -> 3
```

**Interpretation:** The four files contain the same number of records in this controlled example. Equal counts do not prove that the values are identical. A production pipeline should also compare keys, required fields, totals and business rules.

## 11. Schema checks after reading

A format parser only tells you whether it can parse the representation. It does not guarantee that the expected columns exist.

```python
expected_columns = {"order_id", "customer", "amount", "status"}
actual_columns = set(orders_df.columns)

missing_columns = expected_columns - actual_columns
unexpected_columns = actual_columns - expected_columns

print("Missing columns:", missing_columns)
print("Unexpected columns:", unexpected_columns)
```

**Expected output**

```text
Missing columns: set()
Unexpected columns: set()
```

For CSV, inspect `DictReader.fieldnames`. For JSON, validate required keys in each object. For XML, check required elements and attributes. For Parquet, inspect the DataFrame columns and dtypes.

## 12. Common errors and troubleshooting

| Error or issue | Likely cause | Action |
|---|---|---|
| CSV values unexpectedly strings | CSV has no reliable type metadata | Convert and validate fields explicitly |
| `JSONDecodeError` | Malformed JSON text | Inspect the reported location and source payload |
| XML element is `None` | Missing tag or wrong nesting | Check tag names and optional-field rules |
| `ImportError` for Parquet engine | PyArrow/Fastparquet not installed in active environment | Install the engine into the notebook's Python environment |
| File not found | Wrong current directory or filename | Print `Path.cwd()` and verify the file path |
| Duplicate headers | Source schema is malformed | Reject or resolve using a documented source rule |
| Leading zero disappears | Identifier treated as a number | Keep identifiers as strings |

## 13. Production considerations

- Keep the original source file for audit and replay.
- Validate required fields, types and business rules after parsing.
- Treat identifiers as strings when leading zeros or formatting matter.
- Define what happens when optional fields are missing.
- Record source file name, ingestion timestamp and batch identifier.
- Avoid silently dropping malformed records.
- Choose Parquet for appropriate analytical workloads, not merely because it is newer.
- Document schema changes and communicate breaking changes to downstream consumers.

## 14. Day 9 revision

- CSV is a common tabular exchange format; its values need explicit type conversion.
- JSON represents nested objects and arrays and is common in APIs.
- XML represents hierarchical data through elements and attributes.
- Parquet is a typed columnar format widely used in analytics and data lakes.
- `dump/load` work with file objects; `dumps/loads` work with strings.
- Parsing and validation are different steps.
- Day 10 adds exception handling so file-processing pipelines can fail clearly and recover safely.


---

# Beginner Teaching Extension — Day 9

## A. Why formats exist

A data format is an agreed way to represent information so another program can read it. A file extension is only a clue; validate the actual content and schema. The same order can be represented as CSV rows, JSON objects, XML elements or Parquet columns.

## B. CSV: rows and columns

Create orders.csv:

```csv
order_id,customer,amount
1001,Asha,500
1002,Ravi,800
```

Read it:

```python
import csv

with open("orders.csv", "r", newline="", encoding="utf-8") as file:
    reader = csv.DictReader(file)
    for row in reader:
        print(row)
```

Expected output:

```text
{'order_id': '1001', 'customer': 'Asha', 'amount': '500'}
{'order_id': '1002', 'customer': 'Ravi', 'amount': '800'}
```

DictReader uses the header as dictionary keys. CSV values are text, so convert numeric fields deliberately after validating them. newline="" is recommended when using Python's csv module.

## C. JSON: objects and arrays

```json
[
  {"order_id": 1001, "customer": "Asha", "amount": 500},
  {"order_id": 1002, "customer": "Ravi", "amount": 800}
]
```

Read a JSON file:

```python
import json

with open("orders.json", "r", encoding="utf-8") as file:
    orders = json.load(file)

print(type(orders))
print(orders[0]["customer"])
print(orders[1]["amount"])
```

Expected output:

```text
<class 'list'>
Asha
800
```

A JSON array becomes a Python list; each JSON object becomes a dictionary. load() reads from a file-like object; loads() parses a JSON string. Nested data must be inspected rather than assumed to have a flat structure.

## D. Write JSON output

```python
import json

summary = {"record_count": 2, "total_amount": 1300}
with open("summary.json", "w", encoding="utf-8") as file:
    json.dump(summary, file, indent=2)

print("JSON file written")
```

Expected output: JSON file written. dump() writes an object to a file; dumps() returns JSON text as a string.

## E. XML: nested tags and attributes

```xml
<orders>
  <order id="1001">
    <customer>Asha</customer>
    <amount>500</amount>
  </order>
</orders>
```

Parse XML:

```python
import xml.etree.ElementTree as ET

root = ET.fromstring("<orders><order id='1001'><customer>Asha</customer><amount>500</amount></order></orders>")
for order in root.findall("order"):
    print(order.get("id"))
    print(order.findtext("customer"))
    print(order.findtext("amount"))
```

Expected output:

```text
1001
Asha
500
```

fromstring() parses XML text. findall() finds matching child elements. get() reads an attribute. findtext() reads element text. The amount is text until it is converted and validated.

## F. Parquet: columnar analytics format

Parquet stores data by column. Analytics engines can often read only the columns they need and use compression and encoding. It is not intended for manual editing in a text editor.

Install dependencies:

```bash
python -m pip install pandas pyarrow
```

Write and read a small example:

```python
import pandas as pd

orders = pd.DataFrame({"order_id": [1001, 1002], "amount": [500, 800]})
orders.to_parquet("orders.parquet", index=False)
loaded = pd.read_parquet("orders.parquet")
print(loaded)
```

Expected table:

```text
   order_id  amount
0      1001     500
1      1002     800
```

A compatible engine such as PyArrow or fastparquet is required. Formatting may vary by Pandas version, but the rows and values should match.

## G. Compare formats

| Format | Structure | Human readability | Common use |
|---|---|---|---|
| CSV | Flat rows and columns | High | Simple exports and interchange |
| JSON | Objects, arrays and nested values | High | APIs and events |
| XML | Nested elements and attributes | Medium | Enterprise integrations |
| Parquet | Columnar typed storage | Low in raw form | Analytics and data lakes |

No format is always best. Consider schema complexity, interoperability, query pattern, size, compression and which systems must read it.

## H. Mini-project: Multi-Format Data Reader

1. Read two CSV records with DictReader.
2. Read a JSON array and inspect its first record.
3. Parse the XML order and retrieve its ID and amount.
4. Convert equivalent records into a Pandas DataFrame.
5. Write to Parquet and read it back.
6. Compare record counts and totals across representations.

For the two example orders, expected count is 2 and total amount is 1300. Ensure values are numeric before adding them; adding text values can concatenate rather than perform arithmetic.

## I. Common mistakes

- Assuming CSV numbers are automatically numeric.
- Confusing JSON text with a Python dictionary before parsing.
- Treating XML attributes and child elements as identical.
- Assuming Parquet can be read without a compatible engine.
- Trusting an extension without validating the file.
- Ignoring missing columns, extra fields or schema changes.

## J. Student readiness check

The student should explain what each format looks like, read one example of each, describe how parsing maps into Python structures, and justify when CSV, JSON, XML or Parquet would be selected.
