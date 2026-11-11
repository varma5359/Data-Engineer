# Day 35 Project Workbook — TechStore Data Cleaning Framework

This workbook is a guided lab. Run every cell in order. Keep your raw dataset unchanged.

## Lab map
`Create input → Profile → Define rules → Apply rules → Collect all reject reasons → Split → Reconcile → Save → Reload → Test`

## Milestone 1 — Project folder
Create `day35_cleaning_lab` with `data/raw`, `data/accepted`, `data/rejected`, `reports` and `notebooks`. Never save curated output over raw input.

## Milestone 2 — Create and preserve raw data
```python
import pandas as pd
raw_orders = pd.DataFrame({
    "order_id": ["O201", "O202", "O203", "O204", "O204", "O205", "O206"],
    "customer_id": ["C21", "C22", "C23", None, None, "C25", "C26"],
    "amount": ["1500", "900", "500", "700", "700", "-90", "bad-value"],
    "status": [" PAID ", "paid", "PAID", "paid", "paid", "refund", "pending"]
})
raw_orders.to_csv("raw_orders.csv", index=False)
```
There are seven input rows. Explain the defect in each row before proceeding.

## Milestone 3 — Profile before cleaning
```python
print("Shape:", raw_orders.shape)
print("Nulls:\n", raw_orders.isna().sum())
print("Duplicate IDs:", raw_orders["order_id"].duplicated().sum())
print("Statuses:\n", raw_orders["status"].value_counts(dropna=False))
print("Types:\n", raw_orders.dtypes)
```
Record the baseline. Profiling describes; it does not automatically fix.

## Milestone 4 — Working copy and normalization
```python
work = raw_orders.copy()
work["status"] = work["status"].str.strip().str.lower()
work["amount_num"] = pd.to_numeric(work["amount"], errors="coerce")
work[["order_id", "amount", "amount_num", "status"]]
```
Explain why original `amount` remains present: it is evidence of the source value.

## Milestone 5 — Collect every rejection reason (not overwrite)
```python
reasons = [[] for _ in range(len(work))]
```
This creates one empty list per row.

Required order ID:
```python
for i in range(len(work)):
    if pd.isna(work.loc[i, "order_id"]):
        reasons[i].append("missing_order_id")
```

Required customer ID:
```python
for i in range(len(work)):
    if pd.isna(work.loc[i, "customer_id"]):
        reasons[i].append("missing_customer_id")
```

Amount conversion:
```python
for i in range(len(work)):
    if pd.isna(work.loc[i, "amount_num"]):
        reasons[i].append("invalid_amount")
```

Negative amount (contract for this exercise: order amount cannot be negative; refunds are separate):
```python
for i in range(len(work)):
    if pd.notna(work.loc[i, "amount_num"]) and work.loc[i, "amount_num"] < 0:
        reasons[i].append("negative_amount")
```

Duplicate order ID (source contract says order_id uniquely identifies an order):
```python
duplicate_mask = work["order_id"].duplicated(keep="first")
for i in range(len(work)):
    if duplicate_mask.iloc[i]:
        reasons[i].append("duplicate_order_id")
```

Attach all reasons:
```python
work["reject_reasons"] = [";".join(row_reasons) for row_reasons in reasons]
work[["order_id", "customer_id", "amount", "reject_reasons"]]
```
An empty string means no rule detected a rejection. Multiple reasons remain separated by semicolons.

## Milestone 6 — Split and reconcile
```python
accepted = work[work["reject_reasons"] == ""].copy()
rejected = work[work["reject_reasons"] != ""].copy()

print("Input:", len(raw_orders))
print("Accepted:", len(accepted))
print("Rejected:", len(rejected))
print("Reconciled:", len(raw_orders) == len(accepted) + len(rejected))
```
Expected:
```text
Input: 7
Accepted: 3
Rejected: 4
Reconciled: True
```
Rejected rows: O204 missing customer; second O204 duplicate; O205 negative amount; O206 invalid amount.

## Milestone 7 — Curated output
```python
curated = accepted[["order_id", "customer_id", "amount_num", "status"]].rename(
    columns={"amount_num": "amount"}
)
curated
```
Select columns explicitly so only contract-approved fields flow downstream.

## Milestone 8 — Save and reload
```python
curated.to_csv("accepted_orders.csv", index=False)
rejected.to_csv("rejected_orders.csv", index=False)
curated_reloaded = pd.read_csv("accepted_orders.csv")
rejected_reloaded = pd.read_csv("rejected_orders.csv")
print(len(curated_reloaded), len(rejected_reloaded))
```
Expected: `3 4`. Reloading proves files are readable, not that every business rule is correct.

## Milestone 9 — Tests
```python
assert len(raw_orders) == len(accepted) + len(rejected)
assert accepted["order_id"].isna().sum() == 0
assert accepted["customer_id"].isna().sum() == 0
assert accepted["amount_num"].isna().sum() == 0
assert (accepted["amount_num"] >= 0).all()
assert accepted["order_id"].duplicated().sum() == 0
print("All cleaning checks passed")
```
Expected: `All cleaning checks passed`. Assertions stop the notebook if a condition is false.

## Milestone 10 — Schema drift simulation
```python
partner = raw_orders.rename(columns={"customer_id": "customerId"})
expected = {"order_id", "customer_id", "amount", "status"}
received = set(partner.columns)
print("Missing:", expected - received)
print("Extra:", received - expected)
```
Expected missing `customer_id`, extra `customerId`. Confirm semantics with the supplier before applying a mapping.

## Acceptance checklist
- [ ] raw input is unchanged;
- [ ] profile captured before cleaning;
- [ ] cleaning rules documented and business-approved;
- [ ] all rejection reasons retained;
- [ ] accepted and rejected outputs separate;
- [ ] input = accepted + rejected;
- [ ] assertions pass;
- [ ] schema rename is detected;
- [ ] can explain each row decision.

## Submission evidence
Notebook with executed cells, raw input, accepted output, rejected output, profile/reconciliation report and README (rules, assumptions, limitations, rerun instructions).

## Independent challenge
Add optional `currency`; define behavior for missing currency. Make one record violate two rules and prove both reasons remain visible.
