# Day 34: Data Transformation and Business Rules — Beginner Notebook Class

**Phase:** 4 — Data Processing and Pipelines  
**Time:** 2–3 hours  
**Project:** TechStore Order Transformation Pipeline  
**Level:** Complete beginner / fresher  
**Tools:** Python, CSV, Decimal, datetime, Jupyter Notebook  
**Prerequisites:** Python data types/loops, Day 31 file ingestion, Day 33 database extraction

---

# 0. Today's goal

Extraction brings data into a pipeline. But source data may have inconsistent formatting, invalid values, different status codes, or business rules that need to be applied before analytics.

Today you learn to transform data **without silently changing its meaning**.

Flow:
`Raw input -> Profile -> Parse types -> Validate -> Standardize -> Map business codes -> Derive fields -> Accepted/rejected -> Reconcile`

## 0.1 Business scenario

TechStore receives order lines from different systems:
- status values: "paid", " PAID ", "Shipped";
- amount arrives as text;
- quantity and unit price need a line total;
- unknown status appears after a partner update;
- a shipped order has no shipped timestamp.

The analytics team wants consistent curated records. We need approved rules, evidence and rejection handling.

## 0.2 Learning outcomes

You will be able to:
- distinguish raw, cleaned and curated data;
- profile values before changing them;
- convert strings to integer/date/Decimal;
- normalize only when semantics permit;
- map source statuses to canonical values;
- calculate derived fields;
- validate row and cross-field rules;
- preserve rejected records and reasons;
- reconcile counts and totals;
- build a transformation notebook step by step.

---

# 1. Raw vs transformed data

| Layer | Meaning | Example |
|---|---|---|
| Raw | Original received representation | " paid ", "1250.50" |
| Parsed | Values interpreted as types | Decimal('1250.50') |
| Standardized | Consistent formatting | "PAID" |
| Curated | Business rules applied and validated | Accepted order with canonical status |
| Rejected | Cannot safely meet contract | Invalid amount/status with reason |

**Why preserve raw data?** If a mapping rule changes tomorrow, you can replay original values. If you overwrite raw values, you may lose evidence and make corrections impossible.

---

# 2. Inspect data before transforming

Create a practice list:

```python
raw_statuses = ["paid", " PAID ", "Shipped", "pending", "unknown", None]

for status in raw_statuses:
    print(repr(status))
```

Expected:
```text
'paid'
' PAID '
'Shipped'
'pending'
'unknown'
None
```

repr helps reveal whitespace and None. Profiling first tells us what is actually present.

## 2.1 Count values

```python
from collections import Counter

status_counts = Counter(raw_statuses)
print(status_counts)
```

Expected counts depend on the list: each listed value occurs once.

In real data, use frequency counts to identify unexpected variants before deciding a rule.

---

# 3. Standardize text carefully

## 3.1 Strip whitespace

```python
status = " PAID "
clean_status = status.strip()

print(repr(clean_status))
```
Expected:
```text
'PAID'
```

strip removes whitespace at the beginning/end. It does not remove spaces inside a phrase.

## 3.2 Normalize case

```python
status = "paid"
clean_status = status.strip().upper()

print(clean_status)
```
Expected:
```text
PAID
```

**Important:** Case normalization is appropriate when status codes are case-insensitive. It may not be appropriate for customer names, passwords, product codes or fields whose case has meaning.

## 3.3 Handle None

```python
status = None

if status is None:
    clean_status = None
else:
    clean_status = status.strip().upper()

print(clean_status)
```
Expected:
```text
None
```

Do not call strip on None. A null value is not a string.

---

# 4. Parse numeric values

## 4.1 Convert quantity

```python
quantity_text = "3"
quantity = int(quantity_text)

print(quantity)
print(type(quantity))
```
Expected:
```text
3
<class 'int'>
```

Try invalid value "three": int raises ValueError. Record a reject reason rather than silently converting to 0.

## 4.2 Parse money with Decimal

```python
from decimal import Decimal

price_text = "100.00"
unit_price = Decimal(price_text)

print(unit_price)
print(type(unit_price))
```
Expected:
```text
100.00
<class 'decimal.Decimal'>
```

Do not construct Decimal from a float for money (Decimal(0.1)); prefer source text Decimal("0.10").

## 4.3 Validate ranges

```python
amount = Decimal("-25.00")

if amount < 0:
    print("Reject: amount cannot be negative")
else:
    print("Amount accepted")
```
Expected:
```text
Reject: amount cannot be negative
```

Parsing answers “is this value a Decimal?” Validation answers “is this Decimal allowed by business rules?” They are different.

---

# 5. Parse dates and timestamps

```python
from datetime import date

date_text = "2026-11-10"
order_date = date.fromisoformat(date_text)

print(order_date)
```
Expected:
```text
2026-11-10
```

If source format is 10/11/2026, its meaning may be ambiguous (day/month or month/day). Confirm contract before parsing.

Timestamp timezone matters. Do not compare naive and timezone-aware datetimes. Define UTC/business timezone and convert consistently.

---

# 6. Business mapping: source status to canonical status

## 6.1 Why mapping exists

Partners may use different codes for the same meaning:
- P, PAID, payment_complete;
- S, SHIPPED, dispatched;
- W, WAITING, pending.

A mapping table makes business interpretation explicit and reviewable.

## 6.2 Define approved mapping

```python
status_mapping = {
    "P": "PAID",
    "PAID": "PAID",
    "PAYMENT_COMPLETE": "PAID",
    "S": "SHIPPED",
    "SHIPPED": "SHIPPED",
    "W": "PENDING",
    "WAITING": "PENDING"
}
```

## 6.3 Apply mapping

```python
source_status = " payment_complete "
normalized_status = source_status.strip().upper()

canonical_status = status_mapping.get(normalized_status)

print(canonical_status)
```
Expected:
```text
PAID
```

get returns mapped value or None if unknown. Unknown is not automatically PENDING or PAID.

## 6.4 Unknown code handling

```python
source_status = "REFUNDED"
normalized_status = source_status.strip().upper()

if normalized_status not in status_mapping:
    print("Reject/exception: unknown status", normalized_status)
else:
    print("Mapped:", status_mapping[normalized_status])
```
Expected:
```text
Reject/exception: unknown status REFUNDED
```

A business owner may approve a mapping later. Until then, keep it visible.

---

# 7. Derive business values

## 7.1 Line total

```python
quantity = 3
unit_price = Decimal("100.00")
discount = Decimal("10.00")

gross_amount = quantity * unit_price
net_amount = gross_amount - discount

print("Gross:", gross_amount)
print("Net:", net_amount)
```
Expected:
```text
Gross: 300.00
Net: 290.00
```

Explain:
- gross = 3 units × 100 per unit;
- discount is 10 for this line;
- net is gross minus discount;
- currency and discount unit must be documented.

## 7.2 Validate discount

```python
if discount < 0 or discount > gross_amount:
    print("Reject: discount outside allowed range")
else:
    print("Discount valid")
```
Expected:
```text
Discount valid
```

A discount greater than gross may be invalid unless a specific promotion/credit policy allows it.

## 7.3 Null behavior

If quantity or unit_price is missing, do not silently treat it as zero. Choose:
- reject because required;
- apply documented default;
- preserve null and derive null;
- use a domain-specific rule.

Explain the choice to the business.

---

# 8. Cross-field validation

A row can have individually valid fields but contradictory relationships.

Examples:
- status SHIPPED but shipped_at is null;
- cancelled order has positive captured payment when policy says it should be refunded;
- end_date before start_date;
- quantity positive but line amount negative.

```python
status = "SHIPPED"
shipped_at = None

if status == "SHIPPED" and shipped_at is None:
    print("Reject: shipped order must have shipped_at")
```
Expected:
```text
Reject: shipped order must have shipped_at
```

Cross-field rules must be approved and documented. Do not infer a business rule solely from data patterns.

---

# 9. Transform one complete record

Raw record:
```python
raw_record = {
    "partner_order_id": " P-1001 ",
    "customer_id": "501",
    "quantity": "3",
    "unit_price": "100.00",
    "discount": "10.00",
    "status": " payment_complete "
}
```

Transformation:
```python
partner_order_id = raw_record["partner_order_id"].strip()
customer_id = int(raw_record["customer_id"])
quantity = int(raw_record["quantity"])
unit_price = Decimal(raw_record["unit_price"])
discount = Decimal(raw_record["discount"])

source_status = raw_record["status"].strip().upper()
canonical_status = status_mapping[source_status]

gross_amount = quantity * unit_price
net_amount = gross_amount - discount

curated_record = {
    "partner_order_id": partner_order_id,
    "customer_id": customer_id,
    "quantity": quantity,
    "unit_price": str(unit_price),
    "gross_amount": str(gross_amount),
    "discount": str(discount),
    "net_amount": str(net_amount),
    "status": canonical_status
}

print(curated_record)
```

Expected:
```text
{'partner_order_id': 'P-1001', 'customer_id': 501, 'quantity': 3, 'unit_price': '100.00', 'gross_amount': '300.00', 'discount': '10.00', 'net_amount': '290.00', 'status': 'PAID'}
```

This example assumes the source status maps to PAYMENT_COMPLETE. The mapping key must exist before execution.

---

# 10. Accepted and rejected transformation output

For each raw record:
1. preserve source identity;
2. attempt parsing;
3. apply approved standardization/mapping;
4. validate ranges and cross-field rules;
5. collect all safe error reasons;
6. accepted record goes to curated output;
7. rejected record retains source key, run ID and reason;
8. reconcile counts and business totals.

**Don't silently:**
- convert invalid amount to zero;
- map unknown status to a valid state;
- drop missing records;
- remove duplicates without deterministic rule;
- overwrite raw source.

---

# 11. Reconciliation after transformation

Transformation may change values, so reconciliation should reflect expected rules.

## Count check
`input rows = accepted + rejected`

## Amount check
If no rows are rejected and no discount/adjustment changes total:
`source total = curated total`

If business rules change amounts, explain expected difference:
`curated net = source gross - approved discount`

## Status check
Count source statuses and canonical statuses. Confirm all source codes are mapped or rejected/exceptioned.

## Duplicate check
Validate business key before aggregation. Duplicate input can double totals.

---

# 12. Mini-project: TechStore Order Transformation

## Requirement
Transform partner order rows into a canonical curated format with standardized status, typed IDs, Decimal amount and rejection reasons.

## Milestones
1. Read raw CSV (Day 31).
2. Profile status/amount/customer ID values.
3. Define mapping and business rules.
4. Parse types with explicit errors.
5. Calculate gross/net amounts.
6. Validate required/range/cross-field rules.
7. Write curated accepted output.
8. Write rejects with reasons.
9. Reconcile counts, keys and totals.
10. Document transformation version and business rule owner.

## Project run summary
`input_count, accepted_count, rejected_count, unknown_status_count, invalid_amount_count, duplicate_key_count, transformation_version, status`

A transformation is not complete until the rules are explained and the output is verified.

---

# 13. Debugging clinic

| Symptom | Cause | Correct response |
|---|---|---|
| AttributeError strip on None | Null not handled | Check None before string methods |
| ValueError int | Invalid integer text | Reject with reason |
| Decimal InvalidOperation | Invalid amount | Record conversion error |
| Unknown status | Mapping incomplete | Keep exception; ask owner |
| Wrong total | Duplicate grain or discount rule | Check input keys and formula |
| Date shifted | Ambiguous timezone/format | Confirm source contract |
| Accepted count too low | Schema drift/systemic rule | Inspect rejection distribution |
| Totals don't reconcile | Missing/duplicate/rejected or intended adjustment | Explain and trace each component |

---

# 14. Exercises with answers

1. Difference between raw and curated data?
2. Why profile before normalizing?
3. Why is Decimal used for money?
4. Difference between parsing and validation?
5. What should unknown status do?
6. Why preserve rejected records?
7. What is a cross-field rule?
8. Why deduplicate before aggregation?
9. How can transformation affect reconciliation?
10. What evidence proves a mapping is complete?

**Answer guide**
1. Raw preserves source; curated applies approved rules.
2. Discover actual variants and avoid accidental changes.
3. Decimal supports base-10 financial precision.
4. Parsing interprets type; validation checks business constraints.
5. Exception/reject until mapping is approved.
6. Enables correction, traceability and no silent data loss.
7. Rule involving multiple fields.
8. Duplicate lines inflate sums.
9. Cleaning/mapping/derived values can alter fields/totals by design.
10. Every observed source code mapped or explicitly exceptioned; unknown count monitored.

---

# 15. Day 34 self-check

- [ ] Profile raw values before changing.
- [ ] Normalize text only when approved.
- [ ] Convert int/date/Decimal with error handling.
- [ ] Map source status using explicit mapping.
- [ ] Keep unknown status visible.
- [ ] Calculate gross/net with Decimal.
- [ ] Validate cross-field rules.
- [ ] Separate accepted/rejected records.
- [ ] Reconcile counts and business totals.
- [ ] Complete transformation project and explain every rule.

**Remember:** Transformation is business meaning encoded as reproducible rules. A technically successful script can still produce incorrect business data if those rules are wrong.
