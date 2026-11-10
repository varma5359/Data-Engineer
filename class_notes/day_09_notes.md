# Day 9 — CSV, JSON, XML and Parquet Concepts
**Phase 2: Python for Data Engineering**

## 1. Format choice
CSV is flat text; JSON/XML represent nested structures; Parquet is columnar storage for analytics. Format affects schema, types, compression, memory, and interoperability.

## 2. CSV with Python
```python
import csv
with open("orders.csv", newline="", encoding="utf-8") as file:
    reader = csv.DictReader(file)
    for row in reader:
        print(row["order_id"], row["amount"])
```
Each row is a dictionary; values are strings initially. Convert and validate types deliberately.

## 3. JSON
```python
import json
payload = '{"order_id": 101, "customer": {"id": "C01"}, "amount": 500}'
order = json.loads(payload)
print(order["customer"]["id"], order["amount"])
```
Output: `C01 500`

- `loads`: JSON text → Python object.
- `load`: JSON file → Python object.
- `dumps`: Python object → JSON text.
- `dump`: Python object → JSON file.

## 4. XML
XML contains tags and nesting. Python's `xml.etree.ElementTree` can parse simple documents:
```python
import xml.etree.ElementTree as ET
root = ET.fromstring("<order><id>101</id><amount>500</amount></order>")
print(root.find("id").text, root.find("amount").text)
```
Output: `101 500`

Handle namespaces, missing tags, malformed XML, and external input safely.

## 5. Parquet
Parquet stores columns separately and includes schema information; compression and column selection can reduce analytical I/O. It is suited to large datasets in data lakes/warehouses, not manual editing.

Pandas example (requires pandas and a compatible engine such as pyarrow):
```python
import pandas as pd
df = pd.DataFrame({"order_id": [101, 102], "amount": [500, 750]})
df.to_parquet("orders.parquet", index=False)
loaded = pd.read_parquet("orders.parquet")
print(loaded)
```
Output: two rows with order_id and amount columns.

## 6. Format comparison
| Format | Best fit | Watch out |
|---|---|---|
| CSV | Flat interchange | Weak typing, quoting, null ambiguity |
| JSON | API/nested payload | Schema variation |
| XML | Tag-based integrations | Verbose, namespaces |
| Parquet | Analytics at scale | Requires compatible reader |

## Mini-project — Multi-Format Data Reader
Read a small CSV, parse a JSON record, parse XML, and explain how each maps into a common order schema. Keep source parsing separate from normalization.

## Revision
Always validate types and required fields after parsing. Parsing success does not imply data quality.