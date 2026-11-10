# Day 16: Data Validation and Schema Checking

**Phase:** 2 — Python for Data Engineering  
**Mini-project:** Data Validation Engine  
**Learning time:** 2–3 hours

## 1. Learning outcomes
- Explain why data validation is required.
- Distinguish schema validation from business-rule validation.
- Check required fields, types, nulls, ranges and allowed values.
- Convert data types safely.
- Separate valid and rejected records.
- Understand schema contracts and schema evolution.

## 2. Why validation?
A pipeline can execute without errors and still load incorrect data. Examples: negative quantity, invalid timestamp, duplicate order key or unexpected status. Validation detects these conditions before they contaminate downstream tables.

## 3. Validation categories

| Category | Example check |
|---|---|
| Completeness | Required order_id is present |
| Type | amount is numeric |
| Validity | status belongs to allowed set |
| Range | quantity is positive |
| Uniqueness | order line key is unique |
| Consistency | shipped date is not before order date |
| Referential integrity | customer_id exists in customer data |
| Timeliness | source batch arrived within expected window |

## 4. Schema contract
A data contract describes expected structure and behavior between producer and consumer. It may define field names, types, required/optional status, meaning, allowed values, ownership, freshness and change policy.

Example contract:

| Field | Type | Required | Rule |
|---|---|---|---|
| order_id | string | Yes | Stable identifier |
| customer_id | string | Yes | Must reference customer |
| quantity | integer | Yes | Greater than zero |
| amount | decimal | Yes | Non-negative |
| status | string | Yes | Approved status list |
| updated_at | timestamp | Yes | Valid timezone convention |

A schema change should be reviewed for downstream impact rather than accepted blindly.

## 5. Validate dictionary fields
```python
record = {
    "order_id": "501",
    "quantity": "2",
    "amount": "1250.50",
    "status": "PAID"
}

required_fields = ["order_id", "quantity", "amount", "status"]

missing_fields = []

for field in required_fields:
    if field not in record or record[field] is None:
        missing_fields.append(field)

print(missing_fields)
```

## 6. Convert types safely
```python
raw_amount = "1250.50"

try:
    amount = float(raw_amount)
    print(amount)
except ValueError:
    print("Invalid amount")
```

For monetary values, prefer Decimal or database decimal types when exact decimal arithmetic is required.

## 7. Validate allowed values
```python
allowed_statuses = {"PAID", "PENDING", "CANCELLED"}
status = "PAID"

if status not in allowed_statuses:
    print("Invalid status")
else:
    print("Status accepted")
```

Normalize casing/whitespace only if the contract permits it. Do not silently map unknown statuses to a valid status.

## 8. Validate ranges
```python
quantity = 3
amount = 1200.50

if quantity <= 0:
    print("Invalid quantity")

if amount < 0:
    print("Invalid amount")
```

A range rule must come from business meaning. A refund or credit may legitimately be negative in some datasets.

## 9. Validate date/time
```python
from datetime import datetime

raw_date = "2026-01-15"

try:
    parsed_date = datetime.strptime(raw_date, "%Y-%m-%d")
    print(parsed_date)
except ValueError:
    print("Invalid date format")
```

For timestamps, define timezone expectations. A naive timestamp cannot safely be assumed to be UTC.

## 10. Detect duplicates
```python
order_ids = ["501", "502", "501"]

seen = set()
duplicates = []

for order_id in order_ids:
    if order_id in seen:
        duplicates.append(order_id)
    else:
        seen.add(order_id)

print(duplicates)
```

The real uniqueness key may be composite, e.g. order_id plus line_id. Duplicate detection must match the table grain.

## 11. Separate valid and rejected records
```python
records = [
    {"order_id": "501", "quantity": "2", "status": "PAID"},
    {"order_id": "", "quantity": "1", "status": "PAID"},
    {"order_id": "503", "quantity": "-2", "status": "UNKNOWN"}
]

valid_records = []
rejected_records = []

for record in records:
    errors = []

    if not record["order_id"]:
        errors.append("order_id is required")

    try:
        quantity = int(record["quantity"])
        if quantity <= 0:
            errors.append("quantity must be positive")
    except ValueError:
        errors.append("quantity must be an integer")

    if record["status"] not in {"PAID", "PENDING", "CANCELLED"}:
        errors.append("status is not allowed")

    if errors:
        rejected_records.append({"record": record, "errors": errors})
    else:
        valid_records.append(record)

print("Valid:", valid_records)
print("Rejected:", rejected_records)
```

Preserve original values in the rejection output. Do not discard invalid rows without traceability.

## 12. Validation severity
- Critical: required key missing or schema incompatible; stop the load.
- Record-level: one row invalid; quarantine if policy allows.
- Warning: optional field missing or unusual but acceptable value.
- Monitoring: metric drift or freshness approaching threshold.

Define thresholds with the data owner. Never invent a tolerance for production data.

## 13. Schema evolution
A producer may add, remove or change fields. Changes can be:
- Backward-compatible: old consumers continue to work.
- Forward-compatible: new consumers can handle old data.
- Breaking: consumers must change.

Compatibility depends on serialization format and consumer behavior. Test producer and consumer versions together.

## 14. Mini-project: Data Validation Engine
Create an order input dataset and define a contract before coding.

Validation steps:
1. Check required fields.
2. Validate identifier format.
3. Convert quantity and amount.
4. Check ranges.
5. Check allowed status.
6. Detect duplicate business keys.
7. Validate references when dimension data is available.
8. Produce accepted and rejected outputs.
9. Report counts by error type.
10. Stop or continue according to approved severity policy.

Deliverables: schema table, validation code, valid records, rejected records and run summary.

## 15. Common mistakes
| Mistake | Better practice |
|---|---|
| Treating null as zero | Preserve semantic distinction |
| Checking only column names | Validate types and business rules |
| Removing duplicates blindly | Define business key and winner rule |
| Coercing invalid values silently | Record conversion failures |
| Ignoring schema changes | Version contracts and test consumers |
| Accepting every row-level failure | Define rejection thresholds and ownership |

## 16. Day 16 revision
- Schema checks structure; business validation checks meaning.
- Required fields, types, ranges, uniqueness and references matter.
- Keep rejected records with reasons.
- Schema contracts clarify producer/consumer expectations.
- Schema evolution must be compatibility-tested.
