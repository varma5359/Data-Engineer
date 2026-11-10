# Day 9: Data Formats and File Processing

**Phase:** 2 — Python for Data Engineering  
**Mini-project:** Multi-Format Data Reader  
**Learning time:** 2–3 hours

## 1. Learning outcomes

- Read and write CSV and JSON using standard Python modules.
- Understand XML elements and attributes.
- Explain Parquet and columnar storage.
- Recognize serialization and deserialization.
- Select a format based on producer, consumer and workload needs.
- Validate schemas and handle format-specific problems.

## 2. Why formats matter

A pipeline may receive a CSV export from finance, JSON from an API, XML from an enterprise system and Parquet from a lake. The file extension alone does not guarantee valid content.

A format defines how data is represented and parsed. It does not guarantee that values are correct or that two systems agree on business meaning.

## 3. Serialization and deserialization

- Serialization converts in-memory data into a format suitable for storage or transmission.
- Deserialization parses stored/transmitted data into an in-memory representation.

Example: Python dictionary -> JSON text -> Python dictionary.

## 4. CSV

CSV is suitable for simple tabular exchange. Use the `csv` module rather than manually splitting lines.

```python
import csv

rows = [
    {"order_id": 501, "amount": 1200, "status": "PAID"},
    {"order_id": 502, "amount": 800, "status": "PENDING"}
]

with open("orders.csv", "w", newline="", encoding="utf-8") as file:
    fieldnames = ["order_id", "amount", "status"]
    writer = csv.DictWriter(file, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerows(rows)

print("CSV written")
```

Read it:

```python
import csv

with open("orders.csv", "r", newline="", encoding="utf-8") as file:
    reader = csv.DictReader(file)

    for row in reader:
        print(row)
```

CSV reader values are strings. Convert types explicitly and validate before use.

### CSV edge cases

- Quoted commas: `"Hyderabad, Telangana"`.
- Quotes inside text.
- Empty fields.
- Different delimiters.
- Encoding differences.
- Duplicate or missing headers.
- Leading zeros in identifiers.

## 5. JSON

JSON represents objects and arrays. Python's `json` module handles conversion.

```python
import json

order = {
    "order_id": 501,
    "amount": 1200.50,
    "status": "PAID",
    "items": [{"sku": "P10", "quantity": 2}]
}

json_text = json.dumps(order, indent=2)
print(json_text)
```

Write to file:

```python
with open("order.json", "w", encoding="utf-8") as file:
    json.dump(order, file, indent=2)
```

Read JSON:

```python
with open("order.json", "r", encoding="utf-8") as file:
    loaded_order = json.load(file)

print(loaded_order["order_id"])
print(loaded_order["items"][0]["sku"])
```

- `dump` writes JSON to a file.
- `dumps` returns JSON text.
- `load` reads JSON from a file.
- `loads` parses JSON text.

JSON has no native date type. Dates are commonly represented as strings and parsed under an agreed format/time-zone rule.

## 6. Nested JSON

```python
event = {
    "event_id": "E100",
    "customer": {"id": 1001, "city": "Hyderabad"},
    "products": [
        {"sku": "P1", "qty": 2},
        {"sku": "P2", "qty": 1}
    ]
}

print(event["customer"]["city"])

for product in event["products"]:
    print(product["sku"], product["qty"])
```

For relational analytics, a parent event and repeated child items may need separate tables linked by event ID. Do not flatten arrays without deciding what one output row represents.

## 7. XML

XML uses elements and attributes.

```xml
<order id="501">
  <status>PAID</status>
  <amount currency="INR">1200.50</amount>
</order>
```

Parse with ElementTree:

```python
import xml.etree.ElementTree as ET

root = ET.fromstring("""
<order id="501">
  <status>PAID</status>
  <amount currency="INR">1200.50</amount>
</order>
""")

print(root.attrib["id"])
print(root.find("status").text)
print(root.find("amount").text)
print(root.find("amount").attrib["currency"])
```

Elements contain child nodes and text; attributes describe an element. Real XML may include namespaces, optional elements and schema validation requirements.

## 8. Parquet

Parquet is a columnar binary format designed for analytical workloads.

Row-oriented conceptual layout:
- Row 1: order ID, date, amount, status.
- Row 2: order ID, date, amount, status.

Columnar conceptual layout:
- order_id column.
- date column.
- amount column.
- status column.

When a query needs only amount and date, a columnar reader can avoid reading unrelated columns where the storage layout and query engine support projection.

Benefits:
- Efficient column selection.
- Compression.
- Typed schema.
- Useful statistics and predicate pushdown in compatible engines.
- Common support in Spark and analytical tools.

Considerations:
- Not human-readable in a text editor.
- Schema evolution must be managed.
- Too many tiny files can harm distributed query performance.
- Partitioning should reflect common filters without creating excessive directories.

## 9. Read/write Parquet with Pandas

Install if needed:

```bash
python -m pip install pandas pyarrow
```

Create a DataFrame and write:

```python
import pandas as pd

orders_df = pd.DataFrame([
    {"order_id": 501, "amount": 1200.50, "status": "PAID"},
    {"order_id": 502, "amount": 800.00, "status": "PENDING"}
])

orders_df.to_parquet("orders.parquet", index=False)
print("Parquet saved")
```

Read:

```python
loaded_df = pd.read_parquet("orders.parquet")
print(loaded_df)
print(loaded_df.dtypes)
```

Parquet support requires a compatible engine such as PyArrow or fastparquet. Use the same environment for installation and execution.

## 10. Format comparison

| Format | Structure | Human-readable | Best-known use |
|---|---|---|---|
| CSV | Flat rows | Yes | Simple tabular exchange |
| JSON | Objects/arrays | Yes | APIs and events |
| XML | Elements/attributes | Yes | Structured document exchange |
| Parquet | Columnar typed data | No | Analytics and lake storage |

No format is automatically best. Consider interoperability, schema, size, access pattern, compression, query engine and producer contract.

## 11. Schema validation before transformation

For every incoming format, verify:
- Required fields exist.
- Field names and casing match the contract.
- Types can be parsed.
- Required values are not null.
- Allowed values are respected.
- Nested structures have expected shapes.
- Unexpected columns are handled by policy.
- Record count and file completeness are plausible.

Do not silently cast invalid values to zero or empty strings.

## 12. Mini-project: Multi-Format Data Reader

Create a single logical order dataset and represent it in multiple formats.

### Step 1 — Create records

```python
orders = [
    {"order_id": 501, "amount": 1200.50, "status": "PAID"},
    {"order_id": 502, "amount": 800.00, "status": "PENDING"}
]
```

### Step 2 — Save CSV

```python
import csv

with open("orders.csv", "w", newline="", encoding="utf-8") as file:
    writer = csv.DictWriter(file, fieldnames=orders[0].keys())
    writer.writeheader()
    writer.writerows(orders)
```

### Step 3 — Save JSON

```python
import json

with open("orders.json", "w", encoding="utf-8") as file:
    json.dump(orders, file, indent=2)
```

### Step 4 — Read CSV and convert types

```python
with open("orders.csv", "r", newline="", encoding="utf-8") as file:
    csv_rows = list(csv.DictReader(file))

for row in csv_rows:
    row["order_id"] = int(row["order_id"])
    row["amount"] = float(row["amount"])

print(csv_rows)
```

### Step 5 — Read JSON

```python
with open("orders.json", "r", encoding="utf-8") as file:
    json_rows = json.load(file)

print(json_rows)
```

### Step 6 — Compare logical records

```python
print(len(csv_rows), len(json_rows))
print(csv_rows[0]["order_id"] == json_rows[0]["order_id"])
```

Expected: equal record counts and matching first order ID.

### Step 7 — Write Parquet

```python
import pandas as pd

orders_df = pd.DataFrame(json_rows)
orders_df.to_parquet("orders.parquet", index=False)

parquet_df = pd.read_parquet("orders.parquet")
print(parquet_df)
```

### Step 8 — Record validation

Document record count, required fields, type conversion, null count and any rejected rows for each format.

## 13. Common mistakes

| Mistake | Better practice |
|---|---|
| Assuming CSV reader returns numbers | Convert and validate explicitly |
| Using `split(",")` for CSV | Use `csv` module |
| Confusing `json.load` and `json.loads` | File object vs text string |
| Assuming JSON key always exists | Validate optional and required fields |
| Ignoring XML namespaces | Inspect namespace-qualified tags |
| Opening Parquet in text editor | Use a compatible reader |
| Ignoring schema changes | Version and validate the contract |
| Creating thousands of tiny Parquet files | Plan compaction and partitioning |

## 14. Day 9 revision

- Serialization converts objects to stored/transmitted representations.
- CSV is simple tabular text; JSON and XML support nested data.
- Parquet is columnar and suited to analytical access.
- Parsing is not the same as validation.
- Schema, data types and business meaning must be checked.
