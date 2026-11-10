# Day 16 — Data Validation, Schema Checking and Type Conversion
**Phase 2: Python for Data Engineering**

## Why validation?
A script can finish successfully while delivering incorrect data. Validation is a gate before data is published.

## Validation dimensions
- **Schema:** expected columns and types.
- **Completeness:** required fields are present.
- **Uniqueness:** business key is not duplicated.
- **Validity:** values follow business rules.
- **Referential integrity:** referenced customer/product exists.
- **Freshness:** data arrived within its SLA.
- **Reconciliation:** source and destination counts/totals agree.

## Type conversion
```python
amount_text = "1250.50"
amount = float(amount_text)
print(amount)
```
Output: `1250.5`. For financial values use Decimal/database decimal where exact precision matters.

## Required fields
```python
order = {"order_id": 101, "amount": 500, "status": "paid"}
required = ["order_id", "amount", "status"]
missing = [field for field in required if order.get(field) is None]
print(missing)
```
Output: `[]`

## Range/domain rules
```python
amount = 500
status = "paid"
valid = amount >= 0 and status in {"paid", "pending", "refunded"}
print(valid)
```
Output: `True`. Rules must reflect business meaning; a negative amount may represent a refund.

## Reject, quarantine, or accept
- Reject batch for critical schema/contract failure.
- Quarantine isolated invalid rows when policy permits.
- Accept with warning only for documented noncritical conditions.
Always record reason and counts; never silently discard.

## Data contracts
A data contract defines expected schema, meaning, quality, ownership, and compatibility/change expectations between producer and consumer.

## Mini-project — Data Validation Engine
Define rules, validate records, separate accepted/rejected rows, capture rejection reasons, calculate acceptance rate, and fail when agreed thresholds are exceeded.

## Revision
Every rule needs an expectation, test, result, reason, threshold, and action.