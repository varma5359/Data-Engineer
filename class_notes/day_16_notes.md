# Day 16: Data Validation and Schema Checking

**Phase:** 2 — Python for Data Engineering  
**Mini-project:** Data Validation Engine  
**Learning time:** 2–3 hours

## 1. Learning outcomes

- Explain why validation is required before data enters downstream systems.
- Distinguish parsing, schema validation and business-rule validation.
- Define required fields, expected types, allowed values and ranges.
- Convert raw strings safely.
- detect missing values, invalid values and duplicate keys.
- Separate accepted and rejected records with clear reasons.
- Produce a validation summary.
- Understand schema contracts and schema evolution.

## 2. Why validate data?

A pipeline can run without a Python exception and still load incorrect data. Examples:

- An order has a missing order ID.
- Quantity is negative.
- Amount contains text such as `"unknown"`.
- Status is not one of the approved values.
- The same order ID appears twice.
- A date is malformed.

Validation catches these problems before they affect dashboards, finance reports or downstream applications.

## 3. Three different checks

| Check | Question | Example |
|---|---|---|
| Parsing | Can the representation be read? | Is the CSV file syntactically readable? |
| Schema validation | Does the structure match the contract? | Are order_id, amount and status present? |
| Business validation | Are the values acceptable? | Is quantity greater than zero? |

Successful parsing does not prove the schema is correct. A correct schema does not prove the business values are valid.

## 4. Define a sample data contract

A data contract documents what the producer promises and what the consumer expects.

| Field | Expected type | Required | Rule |
|---|---|---|---|
| order_id | integer | Yes | Unique and positive |
| customer_id | integer | Yes | Positive identifier |
| amount | decimal | Yes | Greater than or equal to zero |
| quantity | integer | Yes | Greater than zero |
| status | string | Yes | PAID, PENDING or CANCELLED |
| city | string | No | If supplied, must not be blank |

A contract should also describe field meaning, ownership, freshness, null behavior and how breaking changes are communicated.

## 5. Create raw incoming records

Real file values often arrive as strings, even when they represent numbers.

```python
raw_orders = [
    {"order_id": "1001", "customer_id": "201", "amount": "1200.50", "quantity": "2", "status": "PAID", "city": "Hyderabad"},
    {"order_id": "1002", "customer_id": "202", "amount": "850", "quantity": "1", "status": "PENDING", "city": "Chennai"},
    {"order_id": "", "customer_id": "203", "amount": "400", "quantity": "1", "status": "PAID", "city": "Pune"},
    {"order_id": "1004", "customer_id": "204", "amount": "unknown", "quantity": "1", "status": "PAID", "city": "Pune"},
    {"order_id": "1005", "customer_id": "205", "amount": "-50", "quantity": "1", "status": "PAID", "city": "Chennai"},
    {"order_id": "1006", "customer_id": "206", "amount": "700", "quantity": "0", "status": "PAID", "city": "Hyderabad"},
    {"order_id": "1007", "customer_id": "207", "amount": "900", "quantity": "1", "status": "UNKNOWN", "city": "Mumbai"},
    {"order_id": "1001", "customer_id": "208", "amount": "300", "quantity": "1", "status": "PAID", "city": "Delhi"}
]

print("Incoming records:", len(raw_orders))
print(raw_orders[0])
```

**Expected output**

```text
Incoming records: 8
{'order_id': '1001', 'customer_id': '201', 'amount': '1200.50', 'quantity': '2', 'status': 'PAID', 'city': 'Hyderabad'}
```

This sample intentionally contains missing, malformed, negative, zero, unexpected and duplicate values.

## 6. Check required fields

```python
required_fields = ["order_id", "customer_id", "amount", "quantity", "status"]

record = raw_orders[0]
missing_fields = []

for field in required_fields:
    if field not in record or record[field] is None or str(record[field]).strip() == "":
        missing_fields.append(field)

print("Missing fields:", missing_fields)
```

**Expected output**

```text
Missing fields: []
```

**Explanation**

- `field not in record` detects an absent key.
- `is None` detects a Python null value.
- `strip() == ""` detects an empty or whitespace-only string.

A field that is present with an empty string is not necessarily valid.

## 7. Detect a missing required field

```python
record_with_missing_id = raw_orders[2]
missing_fields = []

for field in required_fields:
    if field not in record_with_missing_id or record_with_missing_id[field] is None or str(record_with_missing_id[field]).strip() == "":
        missing_fields.append(field)

print("Missing fields:", missing_fields)
```

**Expected output**

```text
Missing fields: ['order_id']
```

The row must be rejected or quarantined because the order identifier is required.

## 8. Convert types safely

### 8.1 Convert an integer

```python
raw_quantity = "2"

try:
    quantity = int(raw_quantity)
    print(quantity)
    print(type(quantity).__name__)
except ValueError:
    print("Quantity is not a valid integer")
```

**Expected output**

```text
2
int
```

### 8.2 Invalid conversion

```python
raw_quantity = "two"

try:
    quantity = int(raw_quantity)
    print(quantity)
except ValueError:
    print("Quantity is not a valid integer")
```

**Expected output**

```text
Quantity is not a valid integer
```

### 8.3 Convert money carefully

For financial amounts, Python's `Decimal` avoids some binary floating-point representation surprises.

```python
from decimal import Decimal, InvalidOperation

raw_amount = "1200.50"

try:
    amount = Decimal(raw_amount)
    print(amount)
    print(type(amount).__name__)
except InvalidOperation:
    print("Amount is invalid")
```

**Expected output**

```text
1200.50
Decimal
```

Do not create a Decimal from a float if you need exact decimal input semantics; convert from the original text when possible.

## 9. Validate one record step by step

```python
record = raw_orders[0]
errors = []

if not record.get("order_id"):
    errors.append("order_id is required")

try:
    order_id = int(record["order_id"])
    if order_id <= 0:
        errors.append("order_id must be positive")
except (ValueError, TypeError):
    errors.append("order_id must be an integer")

try:
    amount = Decimal(record["amount"])
    if amount < 0:
        errors.append("amount cannot be negative")
except (InvalidOperation, TypeError):
    errors.append("amount must be a valid decimal")

print("Errors:", errors)
```

**Expected output**

```text
Errors: []
```

This validates one record. We will apply the same rules to every incoming record and collect the reasons for rejection.

## 10. Validate allowed values

```python
allowed_statuses = {"PAID", "PENDING", "CANCELLED"}

status = "UNKNOWN"

if status not in allowed_statuses:
    print("Invalid status:", status)
else:
    print("Status accepted")
```

**Expected output**

```text
Invalid status: UNKNOWN
```

Allowed-value rules should come from a documented source contract. Do not silently map unknown statuses to PAID or another valid status.

## 11. Validate ranges

```python
quantity = 0
amount = Decimal("500")

if quantity <= 0:
    print("Quantity must be greater than zero")

if amount < 0:
    print("Amount cannot be negative")
```

**Expected output**

```text
Quantity must be greater than zero
```

The amount is valid because it is not negative. Range rules should reflect the business definition.

## 12. Build a row-by-row validation engine

We will now validate the complete sample. The engine will:

- Check required fields.
- Convert order_id, customer_id, quantity and amount.
- Check positive identifiers and quantity.
- Check non-negative amount.
- Check allowed status.
- Keep valid records in an accepted list.
- Keep invalid records and their reasons in a rejected list.

### 12.1 Initialize result containers

```python
from decimal import Decimal, InvalidOperation

accepted_records = []
rejected_records = []
seen_order_ids = set()
allowed_statuses = {"PAID", "PENDING", "CANCELLED"}

print("Validation engine initialized")
```

**Expected output**

```text
Validation engine initialized
```

### 12.2 Loop through incoming records

```python
for record in raw_orders:
    errors = []
    cleaned_record = {}

    required_fields = ["order_id", "customer_id", "amount", "quantity", "status"]

    for field in required_fields:
        if field not in record or record[field] is None or str(record[field]).strip() == "":
            errors.append(field + " is required")

    if not errors:
        try:
            cleaned_record["order_id"] = int(record["order_id"])
        except ValueError:
            errors.append("order_id must be an integer")

        try:
            cleaned_record["customer_id"] = int(record["customer_id"])
        except ValueError:
            errors.append("customer_id must be an integer")

        try:
            cleaned_record["quantity"] = int(record["quantity"])
        except ValueError:
            errors.append("quantity must be an integer")

        try:
            cleaned_record["amount"] = Decimal(record["amount"])
        except InvalidOperation:
            errors.append("amount must be a valid decimal")

        cleaned_record["status"] = record["status"]
        cleaned_record["city"] = record.get("city")

    if not errors:
        if cleaned_record["order_id"] <= 0:
            errors.append("order_id must be positive")

        if cleaned_record["customer_id"] <= 0:
            errors.append("customer_id must be positive")

        if cleaned_record["quantity"] <= 0:
            errors.append("quantity must be greater than zero")

        if cleaned_record["amount"] < 0:
            errors.append("amount cannot be negative")

        if cleaned_record["status"] not in allowed_statuses:
            errors.append("status is not allowed")

        if cleaned_record["order_id"] in seen_order_ids:
            errors.append("duplicate order_id")

    if errors:
        rejected_records.append({
            "source_record": record,
            "errors": errors
        })
    else:
        seen_order_ids.add(cleaned_record["order_id"])
        accepted_records.append(cleaned_record)

print("Accepted:", len(accepted_records))
print("Rejected:", len(rejected_records))
```

**Expected output**

```text
Accepted: 2
Rejected: 6
```

**Explanation**

- Each source record receives a fresh error list.
- Required fields are checked before conversion.
- Conversion failures are collected as reasons.
- Business rules run only when conversions succeeded.
- Duplicate detection uses a set of previously accepted IDs.
- A record is added to exactly one output collection.

### 12.3 Inspect accepted records

```python
for record in accepted_records:
    print(record)
```

**Expected output**

Two cleaned records: order 1001 and order 1002. Numeric text has been converted to integers and Decimal.

### 12.4 Inspect rejected reasons

```python
for rejected in rejected_records:
    print(
        rejected["source_record"].get("order_id"),
        "->",
        "; ".join(rejected["errors"])
    )
```

**Expected output**

```text
 -> order_id is required
1004 -> amount must be a valid decimal
1005 -> amount cannot be negative
1006 -> quantity must be greater than zero
1007 -> status is not allowed
1001 -> duplicate order_id
```

The rejected record with a blank order ID displays an empty value before the arrow.

## 13. Understand duplicate detection

A duplicate business key can mean:
- The same source record was delivered twice.
- A retry inserted the same record again.
- The business key is not actually unique.
- Two versions of a record exist and need a versioning rule.

Our simple engine keeps the first valid occurrence and rejects a later occurrence. Production systems may need source timestamps, version numbers or a deterministic latest-record rule.

## 14. Create a validation summary

```python
total_records = len(raw_orders)
accepted_count = len(accepted_records)
rejected_count = len(rejected_records)

print("VALIDATION SUMMARY")
print("Total records:", total_records)
print("Accepted records:", accepted_count)
print("Rejected records:", rejected_count)
print("Counts reconcile:", total_records == accepted_count + rejected_count)
```

**Expected output**

```text
VALIDATION SUMMARY
Total records: 8
Accepted records: 2
Rejected records: 6
Counts reconcile: True
```

The reconciliation check confirms that every source row was assigned to either accepted or rejected output.

## 15. Count rejection reasons

```python
reason_counts = {}

for rejected in rejected_records:
    for error in rejected["errors"]:
        reason_counts[error] = reason_counts.get(error, 0) + 1

for reason, count in reason_counts.items():
    print(reason, ":", count)
```

**Expected output:** A count for each rejection reason present in this sample, including required ID, invalid amount, negative amount, invalid quantity, invalid status and duplicate ID.

The order of dictionary output follows the order in which reasons were first encountered. One record may have more than one rejection reason.

## 16. Save accepted and rejected records

```python
import json

with open("accepted_orders.json", "w", encoding="utf-8") as file:
    json.dump(accepted_records, file, indent=2, default=str)

with open("rejected_orders.json", "w", encoding="utf-8") as file:
    json.dump(rejected_records, file, indent=2, default=str)

print("Accepted file records:", len(accepted_records))
print("Rejected file records:", len(rejected_records))
```

**Expected output**

```text
Accepted file records: 2
Rejected file records: 6
```

`default=str` converts Decimal values into strings for JSON output. Document the output schema so downstream consumers know the serialized representation.

## 17. Schema checking: missing and unexpected columns

For tabular datasets, compare actual columns with expected columns.

```python
expected_columns = {"order_id", "customer_id", "amount", "quantity", "status"}
actual_columns = {"order_id", "customer_id", "amount", "status", "source_file"}

missing_columns = expected_columns - actual_columns
unexpected_columns = actual_columns - expected_columns

print("Missing:", missing_columns)
print("Unexpected:", unexpected_columns)
```

**Expected output**

```text
Missing: {'quantity'}
Unexpected: {'source_file'}
```

A missing required column may be a breaking schema change. An unexpected column may be harmless or may indicate an upstream change that needs review.

## 18. Data quality dimensions

| Dimension | Meaning | Example |
|---|---|---|
| Completeness | Required values are present | order_id is not null |
| Validity | Values follow accepted rules | status is in allowed list |
| Uniqueness | Key values are not duplicated | order_id unique |
| Consistency | Related fields agree | shipped date after order date |
| Accuracy | Values reflect reality | amount matches source transaction |
| Timeliness | Data arrives when expected | daily file arrives by agreed time |

A pipeline can check some dimensions automatically. Accuracy may require reconciliation against a trusted source.

## 19. Mini-project: Data Validation Engine

### 19.1 Requirement

Before loading daily order records into a warehouse, accept only records that meet the contract and retain rejected records with reasons.

### 19.2 Review the workflow

```text
Raw records
   -> Required-field checks
   -> Type conversion
   -> Range and allowed-value checks
   -> Duplicate-key checks
   -> Accepted records + Rejected records
   -> Validation summary
```

### 19.3 Verify final project metrics

```python
print("PROJECT: DATA VALIDATION ENGINE")
print("Input:", len(raw_orders))
print("Accepted:", len(accepted_records))
print("Rejected:", len(rejected_records))
print("Accepted + rejected:", len(accepted_records) + len(rejected_records))
```

**Expected output**

```text
PROJECT: DATA VALIDATION ENGINE
Input: 8
Accepted: 2
Rejected: 6
Accepted + rejected: 8
```

### 19.4 What should happen in production?

- Store the rejected record and the reason without exposing sensitive data unnecessarily.
- Alert the data owner when rejection rates exceed a threshold.
- Preserve source file and batch identifiers for traceability.
- Do not load rejected records into trusted analytical tables.
- Define whether the entire batch fails or valid records can proceed.
- Version the data contract and communicate breaking changes.

## 20. Common mistakes

| Mistake | Why it is risky | Better approach |
|---|---|---|
| Convert before checking blank values | Conversion raises exceptions | Check required fields first |
| Treat missing amount as zero | Changes business meaning | Apply an agreed missing-value policy |
| Silently drop invalid rows | Data loss is hidden | Keep rejected rows with reasons |
| Check duplicates on the wrong field | Valid rows may be rejected | Confirm the business key |
| Accept unknown statuses | Downstream logic may misclassify records | Reject or quarantine unknown values |
| Check only column names | Values may still be invalid | Validate types and business rules |
| Ignore schema changes | Consumers may break unexpectedly | Version contracts and assess impact |

## 21. Day 16 revision

- Parsing, schema checks and business checks are separate.
- Define a contract before writing validation code.
- Convert raw values safely and capture errors.
- Check required fields, ranges, allowed values and duplicate keys.
- Keep accepted and rejected records separately.
- Record rejection reasons and reconcile input counts.
- Day 17 teaches how to test transformation and validation logic automatically.
