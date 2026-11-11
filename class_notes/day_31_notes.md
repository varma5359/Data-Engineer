# Day 31: File-Based Data Ingestion — Beginner Notebook Class

**Phase:** 4 — Data Processing and Pipelines  
**Time:** 2–3 hours  
**Project:** TechStore Partner File Ingestion  
**Level:** Complete beginner / fresher  
**Tools:** Python, Jupyter Notebook or VS Code, CSV and JSON  
**Prerequisites:** Python variables, lists, dictionaries, loops, file operations and exceptions

---

# 0. Today's outcome: build a file ingestion pipeline

A company does not receive all data from databases. Partners, vendors, applications and business teams often send files.

Examples:
- payment provider sends settlement CSV;
- logistics partner sends delivery JSON;
- store teams upload daily inventory;
- marketing partner sends campaign records.

A file arriving is not the same as data being safely ingested. We must identify it, inspect it, validate its structure and values, process it without losing records, and keep evidence of what happened.

Today's flow:

`Partner folder -> Discover files -> Inspect -> Read -> Validate -> Accept/reject -> Write output -> Reconcile -> Run summary`

## 0.1 Learning outcomes

By the end, you should be able to:
1. Explain file ingestion and its business purpose.
2. distinguish file discovery, reading, validation and processing;
3. understand CSV headers, rows, delimiters, encoding and JSON structure;
4. read one file and then multiple files;
5. handle missing/empty/malformed files safely;
6. validate required columns, data types, keys and values;
7. avoid duplicate ingestion on reruns;
8. create accepted and rejected outputs;
9. reconcile input, accepted and rejected counts;
10. explain a file-ingestion project in an interview.

---

# 1. Understand the incoming-file contract

Suppose TechStore receives partner order files in a folder. The agreed CSV contract is:

| Field | Required? | Meaning | Example |
|---|---|---|---|
| partner_order_id | Yes | Partner's stable order key | P-1001 |
| customer_id | Yes | TechStore customer key | 501 |
| order_date | Yes | Order business date | 2026-11-10 |
| status | Yes | Partner order state | PAID |
| amount | Yes | Order amount | 1250.50 |

Before reading code, ask:
- Who produces this file?
- What filename pattern identifies it?
- Is one file one day, one partner, or one batch?
- What encoding/delimiter is used?
- Is header required?
- Which key identifies a record?
- What should happen to invalid records?
- Can the same file arrive twice?
- Can a file be corrected/replaced?
- Who owns schema changes?

**Mentor note:** These are data-contract questions. Python cannot decide business meaning for us.

---

# 2. Create a practice workspace

In Windows CMD:

```bat
mkdir day31_file_ingestion
cd day31_file_ingestion
mkdir input
mkdir accepted
mkdir rejected
mkdir archive
mkdir notebook
```

Folder meaning:

| Folder | Purpose |
|---|---|
| input | Files waiting to be processed |
| accepted | Valid records/output |
| rejected | Invalid records with reasons |
| archive | Original files after successful processing |
| notebook | Learning notebook |

Create `notebook/day31_file_ingestion.ipynb`.

## 2.1 Create a small sample CSV

Create `input/partner_orders_2026-11-10.csv` in VS Code. Type exactly:

```csv
partner_order_id,customer_id,order_date,status,amount
P-1001,501,2026-11-10,PAID,1250.50
P-1002,502,2026-11-10,SHIPPED,800.00
P-1003,503,2026-11-10,PENDING,250.00
```

Save the file as UTF-8. Check that the filename ends in .csv, not .csv.txt.

**What is a CSV?**
- Comma-separated values is a text representation of rows and columns.
- First row is usually a header.
- Each following line represents a record.
- A comma separates fields, unless the contract defines another delimiter.
- CSV does not inherently enforce integer, decimal or date types.

---

# 3. Discover files in a directory

## 3.1 List files

Notebook cell:

```python
from pathlib import Path

input_folder = Path("../input")
print(input_folder.exists())
print(list(input_folder.iterdir()))
```

Expected: True, followed by the sample CSV path (exact display may vary by operating system).

Explanation:
- Path represents a filesystem path.
- exists() checks whether the path exists.
- iterdir() lists immediate directory entries.
- list() materializes the iterator for display.

## 3.2 Find only CSV files

```python
csv_files = list(input_folder.glob("*.csv"))

print("CSV files found:", len(csv_files))
for file_path in csv_files:
    print(file_path.name)
```

Expected:
```text
CSV files found: 1
partner_orders_2026-11-10.csv
```

**Why glob?** The folder may contain README, temporary files or unrelated formats. A pipeline should select files according to an agreed naming/format contract.

## 3.3 No files is not automatically an error

```python
if len(csv_files) == 0:
    print("No CSV files are waiting")
else:
    print("Files are ready for processing")
```

The business schedule decides whether an empty folder is normal or an incident. For a daily partner feed expected by 07:00, no file may mean a missing delivery.

---

# 4. Read a CSV correctly

## 4.1 Use csv.DictReader

```python
import csv

file_path = csv_files[0]

with open(file_path, "r", newline="", encoding="utf-8") as file:
    reader = csv.DictReader(file)
    rows = list(reader)

print("Records read:", len(rows))
print(rows[0])
```

Expected:
```text
Records read: 3
{'partner_order_id': 'P-1001', 'customer_id': '501', 'order_date': '2026-11-10', 'status': 'PAID', 'amount': '1250.50'}
```

**Important discovery:** CSV values arrive as strings. Even customer_id and amount are text until we convert them.

## 4.2 Explain the reader

- `open` opens the file for reading.
- `encoding="utf-8"` interprets bytes as UTF-8 text.
- `newline=""` lets the CSV library handle newline conventions.
- `csv.DictReader` uses the header as dictionary keys.
- `list(reader)` reads every record into memory (fine for three sample rows; not suitable for huge files).
- `rows[0]` means first data record, not the header.

## 4.3 Inspect fields

```python
print(reader.fieldnames)
print(type(rows[0]["customer_id"]))
print(type(rows[0]["amount"]))
```

Expected:
```text
['partner_order_id', 'customer_id', 'order_date', 'status', 'amount']
<class 'str'>
<class 'str'>
```

A CSV reader does not know the business type. Our validation/conversion layer must establish it.

---

# 5. Validate the file structure before processing

## 5.1 Required column list

```python
required_columns = [
    "partner_order_id",
    "customer_id",
    "order_date",
    "status",
    "amount"
]

received_columns = reader.fieldnames
missing_columns = [
    column for column in required_columns
    if column not in received_columns
]

print("Missing columns:", missing_columns)
```

Expected:
```text
Missing columns: []
```

If a required column is missing, stop processing that file. Do not create a fake empty column silently.

## 5.2 Extra columns

```python
extra_columns = [
    column for column in received_columns
    if column not in required_columns
]

print("Extra columns:", extra_columns)
```

Extra columns may be allowed by an additive schema policy, but must be observed. Do not blindly discard them if the new field may carry business meaning.

## 5.3 Duplicate header check

```python
print("Header count:", len(received_columns))
print("Unique header count:", len(set(received_columns)))
```

If these counts differ, duplicate column names exist. A dictionary reader can overwrite one duplicate key, so reject or explicitly resolve duplicate headers before reading records.

---

# 6. Convert values with clear rules

## 6.1 Customer ID

```python
customer_id = int(rows[0]["customer_id"])
print(customer_id)
print(type(customer_id))
```

Expected:
```text
501
<class 'int'>
```

Conversion changes a string into an integer. It can fail for values such as "", "five hundred" or "501.2".

## 6.2 Amount — use Decimal for money

```python
from decimal import Decimal

amount = Decimal(rows[0]["amount"])
print(amount)
print(type(amount))
```

Expected:
```text
1250.50
<class 'decimal.Decimal'>
```

**Why not float?** Binary floating-point cannot represent every decimal fraction exactly. For financial values, Decimal is a safer representation when the source and business rules use decimal precision.

## 6.3 Date

```python
from datetime import date

order_date = date.fromisoformat(rows[0]["order_date"])
print(order_date)
print(type(order_date))
```

Expected:
```text
2026-11-10
<class 'datetime.date'>
```

This expects ISO format YYYY-MM-DD. Other date formats need an explicit parser and contract.

---

# 7. Validate each record

A record can have correct columns but invalid values.

## 7.1 Business rules

For our teaching contract:
- partner_order_id cannot be blank;
- customer_id must be a positive integer;
- order_date must be a valid ISO date;
- status must be PAID, SHIPPED or PENDING;
- amount must be a valid Decimal and cannot be negative.

These are sample business rules. A real company must approve its own rules.

## 7.2 Validate one record step by step

```python
record = rows[0]

errors = []

if record["partner_order_id"].strip() == "":
    errors.append("partner_order_id is blank")

try:
    customer_id = int(record["customer_id"])
    if customer_id <= 0:
        errors.append("customer_id must be positive")
except ValueError:
    errors.append("customer_id is not an integer")

try:
    order_date = date.fromisoformat(record["order_date"])
except ValueError:
    errors.append("order_date is not YYYY-MM-DD")

if record["status"] not in ["PAID", "SHIPPED", "PENDING"]:
    errors.append("status is not allowed")

try:
    amount = Decimal(record["amount"])
    if amount < 0:
        errors.append("amount cannot be negative")
except Exception:
    errors.append("amount is not a valid decimal")

print("Errors:", errors)
```

Expected for P-1001:
```text
Errors: []
```

Read this carefully:
- errors begins as an empty list.
- each failed rule adds a reason.
- try/except catches conversion failures so the notebook can record the reason.
- validation does not modify the source record.
- no errors means this record passed these specific rules—not every possible business rule.

---

# 8. Accept and reject records without losing evidence

## 8.1 Add an invalid row for practice

Append this line to a separate test CSV:

```csv
P-1004,abc,2026-11-10,UNKNOWN,-25
```

It contains multiple problems:
- customer_id is not an integer;
- status is not in the allowed set;
- amount is negative.

Never corrupt a real production partner file for a training exercise. Use a copy.

## 8.2 Process records into two lists

```python
accepted_records = []
rejected_records = []

for record in rows:
    errors = []

    if record["partner_order_id"].strip() == "":
        errors.append("partner_order_id is blank")

    try:
        customer_id = int(record["customer_id"])
        if customer_id <= 0:
            errors.append("customer_id must be positive")
    except ValueError:
        errors.append("customer_id is not an integer")

    try:
        order_date = date.fromisoformat(record["order_date"])
    except ValueError:
        errors.append("order_date is not YYYY-MM-DD")

    if record["status"] not in ["PAID", "SHIPPED", "PENDING"]:
        errors.append("status is not allowed")

    try:
        amount = Decimal(record["amount"])
        if amount < 0:
            errors.append("amount cannot be negative")
    except Exception:
        errors.append("amount is not a valid decimal")

    if errors:
        rejected_records.append({
            "source_file": file_path.name,
            "partner_order_id": record.get("partner_order_id", ""),
            "rejection_reason": "; ".join(errors)
        })
    else:
        accepted_records.append({
            "partner_order_id": record["partner_order_id"],
            "customer_id": customer_id,
            "order_date": order_date.isoformat(),
            "status": record["status"],
            "amount": str(amount)
        })

print("Accepted:", len(accepted_records))
print("Rejected:", len(rejected_records))
```

Expected with three valid sample rows and one deliberately invalid row:
```text
Accepted: 3
Rejected: 1
```

**Why separate lists?** Accepted data can continue to the destination. Rejected data retains an explanation for correction. Silently dropping a row makes data loss difficult to investigate.

---

# 9. Write accepted and rejected outputs

## 9.1 Accepted CSV

```python
accepted_path = Path("../accepted/partner_orders_accepted.csv")

with open(accepted_path, "w", newline="", encoding="utf-8") as file:
    writer = csv.DictWriter(
        file,
        fieldnames=[
            "partner_order_id",
            "customer_id",
            "order_date",
            "status",
            "amount"
        ]
    )
    writer.writeheader()
    writer.writerows(accepted_records)

print("Accepted output:", accepted_path)
```

The accepted output has normalized values: customer_id is converted and amount is rendered as a decimal string. CSV itself still stores text.

## 9.2 Rejected CSV

```python
rejected_path = Path("../rejected/partner_orders_rejected.csv")

with open(rejected_path, "w", newline="", encoding="utf-8") as file:
    writer = csv.DictWriter(
        file,
        fieldnames=["source_file", "partner_order_id", "rejection_reason"]
    )
    writer.writeheader()
    writer.writerows(rejected_records)

print("Rejected output:", rejected_path)
```

Keep enough safe evidence to fix the record. Avoid placing unnecessary PII or secrets in reject files; restrict access and retention.

---

# 10. Reconcile ingestion

## 10.1 Count invariant

For row-level validation:

`Input records = Accepted records + Rejected records`

```python
input_count = len(rows)
accepted_count = len(accepted_records)
rejected_count = len(rejected_records)

print("Input:", input_count)
print("Accepted:", accepted_count)
print("Rejected:", rejected_count)
print("Reconciled:", input_count == accepted_count + rejected_count)
```

Expected for four input records:
```text
Input: 4
Accepted: 3
Rejected: 1
Reconciled: True
```

If Reconciled is False, some record was lost or counted incorrectly. Do not mark the run successful.

## 10.2 Key uniqueness

```python
accepted_keys = [
    record["partner_order_id"]
    for record in accepted_records
]

duplicate_count = len(accepted_keys) - len(set(accepted_keys))

print("Duplicate accepted keys:", duplicate_count)
```

A duplicate key may mean partner replay, duplicate rows in one file, or conflicting record versions. The correct action depends on the partner contract. Do not automatically delete duplicates without defining which record wins.

---

# 11. JSON ingestion fundamentals

CSV is tabular. JSON represents objects, arrays and nested structures.

## 11.1 Create sample JSON

Create `input/partner_delivery.json`:

```json
{
  "partner": "FastShip",
  "business_date": "2026-11-10",
  "deliveries": [
    {"delivery_id": "D-001", "order_id": 1001, "status": "DELIVERED"},
    {"delivery_id": "D-002", "order_id": 1002, "status": "IN_TRANSIT"}
  ]
}
```

## 11.2 Read JSON

```python
import json

json_path = Path("../input/partner_delivery.json")

with open(json_path, "r", encoding="utf-8") as file:
    delivery_data = json.load(file)

print(delivery_data["partner"])
print(len(delivery_data["deliveries"]))
print(delivery_data["deliveries"][0])
```

Expected:
```text
FastShip
2
{'delivery_id': 'D-001', 'order_id': 1001, 'status': 'DELIVERED'}
```

Explanation:
- `json.load` parses the file into Python objects.
- JSON object becomes dictionary.
- JSON array becomes list.
- Nested object becomes dictionary inside the list.
- Keys must exist before indexing; missing keys raise KeyError.

## 11.3 Validate JSON structure

```python
required_json_keys = ["partner", "business_date", "deliveries"]

missing_json_keys = [
    key for key in required_json_keys
    if key not in delivery_data
]

print("Missing JSON keys:", missing_json_keys)
print("Deliveries is list:", isinstance(delivery_data["deliveries"], list))
```

Expected:
```text
Missing JSON keys: []
Deliveries is list: True
```

Valid JSON syntax is not the same as a valid business payload. A file can parse but contain wrong/missing keys or wrong types.

---

# 12. Multiple files and file-level failure

A production folder may contain:
- valid CSV;
- empty file;
- malformed CSV;
- unexpected file extension;
- duplicate delivery;
- file still being copied by the partner.

Do not process a file while it is still being written. Use a delivery convention such as temporary filename followed by rename, or a manifest/ready marker, agreed with the producer.

## 12.1 Inspect size

```python
for file_path in csv_files:
    print(file_path.name, file_path.stat().st_size, "bytes")
```

A zero-byte file is different from a file containing a header but zero records. Define expected behavior for each.

## 12.2 File-level vs record-level error

- File-level error: missing required header, malformed structure, wrong encoding, truncated file. Usually stop/quarantine the file.
- Record-level error: one row has invalid amount or status. Depending on contract, reject that row and continue.
- Systemic error: every row fails because the producer changed schema. Stop the batch and alert rather than quietly rejecting everything.

---

# 13. Duplicate file and idempotency

The partner may send the same file twice. If we append blindly, records duplicate.

Possible controls:
- stable file identity (partner + delivery ID/date);
- checksum/hash to detect identical content;
- ingestion ledger storing file name, hash, arrival time, status and run ID;
- target unique key and idempotent upsert;
- explicit correction/version policy.

**Filename alone is not always identity:** a partner can reuse a name with corrected content. Compare checksum/version and contract.

A rerun should be deliberate:
- already successful identical file -> skip or safely replay;
- same file with changed content -> correction/version workflow;
- failed file -> retry after cause is fixed;
- partial processing -> inspect durable state before restart.

---

# 14. Mini-project: TechStore Partner File Ingestion

## Requirement

Every morning, ingest partner order CSV files, validate rows, write accepted and rejected outputs, and generate a run summary.

## Project steps

1. Discover files matching partner_orders_*.csv.
2. Record file name and size.
3. Check required/duplicate headers.
4. Read rows and validate each record.
5. Normalize customer ID, date and amount.
6. Write accepted records.
7. Write rejected records with reasons.
8. Reconcile input = accepted + rejected.
9. Check accepted business key uniqueness.
10. Record file status and run summary.
11. Do not archive/publish as successful unless validations pass.

## Run summary example (values depend on files)

```python
run_summary = {
    "files_discovered": len(csv_files),
    "records_read": input_count,
    "accepted": accepted_count,
    "rejected": rejected_count,
    "reconciled": input_count == accepted_count + rejected_count,
    "duplicate_keys": duplicate_count,
    "status": "SUCCESS"
}

print(run_summary)
```

Expected shape:
```text
{'files_discovered': 1,
 'records_read': 4,
 'accepted': 3,
 'rejected': 1,
 'reconciled': True,
 'duplicate_keys': 0,
 'status': 'SUCCESS'}
```

**Status rule:** SUCCESS only if file-level checks pass, reconciliation passes, and duplicate policy passes. A run with row rejects may be SUCCESS_WITH_REJECTS if the contract allows it.

---

# 15. Debugging clinic

| Symptom | Cause to investigate | Correct response |
|---|---|---|
| File not found | Wrong working directory/path | Print Path.cwd(), inspect folder |
| Zero files discovered | Wrong glob/pattern or delivery absent | Check actual name and schedule |
| UnicodeDecodeError | Encoding mismatch | Confirm producer encoding; don't guess silently |
| KeyError in row | Header missing/typo | Validate headers before row processing |
| ValueError converting ID | Invalid text | Reject with clear reason |
| JSONDecodeError | Invalid/truncated JSON | Quarantine file; request corrected delivery |
| Accepted count zero | Schema drift or all invalid | Inspect reject reasons; stop/alert if systemic |
| Duplicate keys | Replayed/conflicting partner records | Apply documented dedup/version rule |
| Partial output | Crash during write | Temp output, validate, publish atomically |
| Rerun duplicates | Append or no idempotency | File ledger/unique key/replay-safe output |

---

# 16. Exercises with answers

1. Why does CSV reader return customer_id as string?
2. What is the difference between missing column and invalid value?
3. Why should a malformed JSON file be handled at file level?
4. How do you prove no input row disappeared?
5. Why isn't filename alone always a unique delivery identity?
6. What does a duplicate partner_order_id mean?
7. What is the danger of accepting a file while it is still uploading?
8. When should a pipeline stop rather than reject every row?
9. What evidence should a rejected record contain?
10. How should a rerun behave for an identical successful file?

**Answer guide**
1. CSV is text; type rules are external.
2. Missing column breaks the file contract; invalid value breaks one record's rule.
3. The parser cannot reliably produce records from malformed structure.
4. Input count = accepted + rejected, plus file-level record accounting.
5. Partners can reuse names or send corrected content.
6. Replay, duplicate, or conflict—business contract determines.
7. Truncated/partial data may be ingested.
8. When schema/systemic error affects the whole batch or correctness cannot be guaranteed.
9. Source file, stable record key, rule/reason, run ID and safe context.
10. Skip or safely replay without duplicate effects, according to ledger/output policy.

---

# 17. Day 31 self-check

- [ ] Create a folder and sample CSV.
- [ ] Discover only matching files.
- [ ] Read CSV and explain why values are strings.
- [ ] Validate required headers and duplicate headers.
- [ ] Convert integer, Decimal and date with error handling.
- [ ] Accept valid rows and reject invalid rows with reasons.
- [ ] Read nested JSON and validate its structure.
- [ ] Reconcile input/accepted/rejected counts.
- [ ] Explain duplicate-file and rerun behavior.
- [ ] Complete partner-file project and defend it in interview.

**Remember:** File ingestion is not “read CSV and save CSV.” It is controlled delivery discovery, contract validation, record processing, evidence, safe publication and recovery.
