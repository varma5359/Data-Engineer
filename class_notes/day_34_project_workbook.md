# Day 34 Project Workbook — TechStore Transformation Pipeline

## Project goal
Transform raw order records into curated accepted records; reject invalid rows with explicit reasons; reconcile counts and amounts.

## Step 0 — Input fixture
Create a Python cell:
```python
from decimal import Decimal
raw_records = [
    {"order_id":"P-1","customer_id":"501","quantity":"2","unit_price":"100.00","discount":"10.00","status":"paid"},
    {"order_id":"P-2","customer_id":"502","quantity":"1","unit_price":"50.00","discount":"0.00","status":"shipped"},
    {"order_id":"P-3","customer_id":"abc","quantity":"2","unit_price":"20.00","discount":"0.00","status":"unknown"},
    {"order_id":"P-4","customer_id":"504","quantity":"1","unit_price":"-5.00","discount":"0.00","status":"pending"}
]
```
Four rows: two expected valid, two deliberately invalid.

## Step 1 — Profile source status
```python
from collections import Counter
print(Counter(record["status"] for record in raw_records))
```
Expected: paid, shipped, unknown, pending each appear once. Profiling identifies codes before mapping.

## Step 2 — Define approved mapping
```python
status_mapping = {
    "PAID": "PAID",
    "SHIPPED": "SHIPPED",
    "PENDING": "PENDING"
}
```
Unknown status remains unknown; do not guess a mapping.

## Step 3 — Transform and validate
```python
from decimal import Decimal, InvalidOperation

accepted = []
rejected = []

for raw in raw_records:
    errors = []
    order_id = raw["order_id"].strip()
    status_source = raw["status"].strip().upper()

    try:
        customer_id = int(raw["customer_id"])
        if customer_id <= 0:
            errors.append("customer_id must be positive")
    except ValueError:
        customer_id = None
        errors.append("customer_id is not an integer")

    try:
        quantity = int(raw["quantity"])
        if quantity <= 0:
            errors.append("quantity must be positive")
    except ValueError:
        quantity = None
        errors.append("quantity is not an integer")

    try:
        unit_price = Decimal(raw["unit_price"])
        if unit_price < 0:
            errors.append("unit_price cannot be negative")
    except InvalidOperation:
        unit_price = None
        errors.append("unit_price is invalid")

    try:
        discount = Decimal(raw["discount"])
        if discount < 0:
            errors.append("discount cannot be negative")
    except InvalidOperation:
        discount = None
        errors.append("discount is invalid")

    canonical_status = status_mapping.get(status_source)
    if canonical_status is None:
        errors.append("unknown status")

    if not errors:
        gross = quantity * unit_price
        if discount > gross:
            errors.append("discount exceeds gross amount")

    if errors:
        rejected.append({
            "order_id": order_id,
            "reason": "; ".join(errors)
        })
    else:
        accepted.append({
            "order_id": order_id,
            "customer_id": customer_id,
            "quantity": quantity,
            "unit_price": str(unit_price),
            "gross_amount": str(gross),
            "discount": str(discount),
            "net_amount": str(gross - discount),
            "status": canonical_status
        })

print("Accepted:", len(accepted))
print("Rejected:", len(rejected))
```
Expected:
```text
Accepted: 2
Rejected: 2
```
P-3 fails customer ID and status; P-4 fails negative price. A row can have multiple rejection reasons.

## Step 4 — Inspect curated rows
```python
for record in accepted:
    print(record)
```
Expected business values:
- P-1 gross 200.00, net 190.00, status PAID.
- P-2 gross 50.00, net 50.00, status SHIPPED.

## Step 5 — Reconcile
```python
print("Input:", len(raw_records))
print("Accepted + rejected:", len(accepted) + len(rejected))
print("Reconciled:", len(raw_records) == len(accepted) + len(rejected))
```
Expected: Input 4, Accepted + rejected 4, Reconciled True.

## Step 6 — Check keys and totals
```python
accepted_ids = [record["order_id"] for record in accepted]
print("Unique accepted IDs:", len(set(accepted_ids)))
print("Accepted IDs:", len(accepted_ids))
print("Net amount total:", sum(Decimal(record["net_amount"]) for record in accepted))
```
Expected unique IDs=2 and accepted IDs=2. Net total=240.00 for fixture.

## Step 7 — Write outputs
Use csv.DictWriter with fixed fieldnames for curated and rejected outputs. Always define fieldnames separately so a zero-row output still gets a header. Reopen each file and count data rows.

## Step 8 — Rule-change test
Change the mapping to include a new approved code (for example REFUNDED only after business owner approval). Re-run raw input and compare output. Record mapping version; never edit raw source.

## Acceptance criteria
- [ ] Raw values profiled.
- [ ] Approved mapping documented.
- [ ] Type conversions handle failures.
- [ ] Unknown values remain exceptions.
- [ ] Gross/net calculations correct.
- [ ] Accepted/rejected outputs written.
- [ ] Counts reconcile.
- [ ] Duplicate keys and totals checked.
- [ ] Rule version recorded.
