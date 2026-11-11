# Day 31 Project Workbook — Partner File Ingestion

Follow every milestone. This workbook explains folder setup, input data, execution, expected results, validation and recovery.

## Project goal

Ingest partner CSV files into accepted/rejected outputs. Keep evidence for every record and avoid duplicate effects on rerun.

## Step 0 — Create workspace

In CMD:
```bat
mkdir day31_file_ingestion
cd day31_file_ingestion
mkdir input accepted rejected archive notebook
```

Create `notebook/day31_file_ingestion.ipynb`.

## Step 1 — Create training input

Create `input/partner_orders_2026-11-10.csv`:

```csv
partner_order_id,customer_id,order_date,status,amount
P-1001,501,2026-11-10,PAID,1250.50
P-1002,502,2026-11-10,SHIPPED,800.00
P-1003,503,2026-11-10,PENDING,250.00
P-1004,abc,2026-11-10,UNKNOWN,-25
```

Expected: four data rows, one header. The last row is deliberately invalid for validation practice.

## Step 2 — Discover files

Notebook cell:
```python
from pathlib import Path
import csv
import json
from decimal import Decimal
from datetime import date

input_folder = Path("../input")
csv_files = sorted(input_folder.glob("partner_orders_*.csv"))

print("Files discovered:", len(csv_files))
for item in csv_files:
    print(item.name, item.stat().st_size)
```

Expected:
```text
Files discovered: 1
partner_orders_2026-11-10.csv <file size in bytes>
```
File size is environment-specific.

## Step 3 — Validate file header before records

```python
required = ["partner_order_id", "customer_id", "order_date", "status", "amount"]

with open(csv_files[0], "r", newline="", encoding="utf-8") as file:
    reader = csv.DictReader(file)
    header = reader.fieldnames
    input_rows = list(reader)

missing = [field for field in required if field not in (header or [])]
duplicate_headers = len(header or []) != len(set(header or []))

print("Header:", header)
print("Missing:", missing)
print("Duplicate header:", duplicate_headers)
print("Input records:", len(input_rows))
```

Expected:
```text
Header: ['partner_order_id', 'customer_id', 'order_date', 'status', 'amount']
Missing: []
Duplicate header: False
Input records: 4
```

If missing or duplicate headers exist, stop processing the file and mark file-level failure. Do not proceed to row validation.

## Step 4 — Validate each row

```python
accepted = []
rejected = []

allowed_statuses = {"PAID", "SHIPPED", "PENDING"}

for row in input_rows:
    errors = []

    partner_order_id = row.get("partner_order_id", "").strip()
    if not partner_order_id:
        errors.append("partner_order_id is blank")

    try:
        customer_id = int(row["customer_id"])
        if customer_id <= 0:
            errors.append("customer_id must be positive")
    except ValueError:
        customer_id = None
        errors.append("customer_id is not an integer")

    try:
        order_date = date.fromisoformat(row["order_date"])
    except ValueError:
        order_date = None
        errors.append("order_date is not YYYY-MM-DD")

    status = row["status"].strip()
    if status not in allowed_statuses:
        errors.append("status is not allowed")

    try:
        amount = Decimal(row["amount"])
        if amount < 0:
            errors.append("amount cannot be negative")
    except Exception:
        amount = None
        errors.append("amount is not a valid decimal")

    if errors:
        rejected.append({
            "source_file": csv_files[0].name,
            "partner_order_id": partner_order_id,
            "rejection_reason": "; ".join(errors)
        })
    else:
        accepted.append({
            "partner_order_id": partner_order_id,
            "customer_id": customer_id,
            "order_date": order_date.isoformat(),
            "status": status,
            "amount": str(amount)
        })

print("Accepted:", len(accepted))
print("Rejected:", len(rejected))
```

Expected:
```text
Accepted: 3
Rejected: 1
```

Inspect rejection:
```python
print(rejected[0])
```
Expected reason includes customer_id is not an integer, status is not allowed, and amount cannot be negative. Exact reason order follows the validation code.

## Step 5 — Write accepted and rejected outputs

```python
accepted_file = Path("../accepted/partner_orders_accepted.csv")
rejected_file = Path("../rejected/partner_orders_rejected.csv")

with open(accepted_file, "w", newline="", encoding="utf-8") as file:
    writer = csv.DictWriter(file, fieldnames=list(accepted[0].keys()))
    writer.writeheader()
    writer.writerows(accepted)

with open(rejected_file, "w", newline="", encoding="utf-8") as file:
    writer = csv.DictWriter(file, fieldnames=list(rejected[0].keys()))
    writer.writeheader()
    writer.writerows(rejected)

print("Accepted file:", accepted_file)
print("Rejected file:", rejected_file)
```

For a real pipeline, handle the case where accepted or rejected is empty before using accepted[0]/rejected[0]. Define headers independently so empty output can still contain a header.

## Step 6 — Reopen outputs and reconcile

```python
with open(accepted_file, "r", newline="", encoding="utf-8") as file:
    accepted_check = list(csv.DictReader(file))

with open(rejected_file, "r", newline="", encoding="utf-8") as file:
    rejected_check = list(csv.DictReader(file))

input_count = len(input_rows)
accepted_count = len(accepted_check)
rejected_count = len(rejected_check)

print("Input:", input_count)
print("Accepted output:", accepted_count)
print("Rejected output:", rejected_count)
print("Reconciled:", input_count == accepted_count + rejected_count)
```

Expected:
```text
Input: 4
Accepted output: 3
Rejected output: 1
Reconciled: True
```

## Step 7 — Check duplicate keys

```python
accepted_ids = [row["partner_order_id"] for row in accepted_check]
duplicate_count = len(accepted_ids) - len(set(accepted_ids))
print("Duplicate accepted IDs:", duplicate_count)
```

Expected for this fixture: 0.

If the input includes duplicate IDs, decide whether identical duplicates are replay and differing payloads are conflict. Do not silently choose a record.

## Step 8 — Build run summary

```python
run_summary = {
    "files_discovered": len(csv_files),
    "records_read": input_count,
    "accepted": accepted_count,
    "rejected": rejected_count,
    "reconciled": input_count == accepted_count + rejected_count,
    "duplicate_keys": duplicate_count,
    "status": "SUCCESS_WITH_REJECTS"
}

print(run_summary)
```

Expected:
```text
{'files_discovered': 1, 'records_read': 4, 'accepted': 3, 'rejected': 1, 'reconciled': True, 'duplicate_keys': 0, 'status': 'SUCCESS_WITH_REJECTS'}
```

The status is appropriate only if the business contract allows row-level rejects. If rejects exceed a threshold or are caused by a schema-wide change, mark failure and alert.

## Step 9 — Rerun test

Run all cells again. Confirm:
- accepted output contains exactly 3 rows, not 6;
- rejected output contains exactly 1 row, not 2;
- files are overwritten only under your documented training output policy;
- production reruns would use a file ledger, run ID and safe publication/target idempotency.

## Step 10 — Add JSON file

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

Read and inspect:
```python
json_path = Path("../input/partner_delivery.json")
with open(json_path, "r", encoding="utf-8") as file:
    payload = json.load(file)

print(payload["partner"])
print(len(payload["deliveries"]))
print(payload["deliveries"][0]["delivery_id"])
```

Expected:
```text
FastShip
2
D-001
```

Validate that payload is a dictionary, deliveries exists and is a list, and each delivery has required keys. Valid JSON syntax alone does not guarantee valid data.

## Step 11 — Failure drills

1. Remove status header in a copy: expected file-level rejection.
2. Put `abc` in customer_id: expected record-level rejection.
3. Use malformed JSON: expected JSONDecodeError and quarantine; do not continue as if empty.
4. Duplicate partner_order_id: expected duplicate check catches it.
5. Empty file: decide file-level failure or valid empty delivery per contract.
6. Rename file to .csv.txt: glob should not discover it.
7. Rerun identical file: verify no duplicate output effects.
8. Change file content but reuse name: identify via checksum/version and correction policy.

## Acceptance checklist

- [ ] Correct folder and file discovery.
- [ ] Header/schema validation before records.
- [ ] Record conversion and validation with reason.
- [ ] Accepted and rejected files written.
- [ ] Input = accepted + rejected.
- [ ] Key uniqueness check.
- [ ] JSON parsing/structure check.
- [ ] Empty/malformed/schema-change behavior defined.
- [ ] Rerun behavior tested.
- [ ] Student can explain every step and output.

**Project completion means records are accounted for, errors are visible and reruns are safe.**
