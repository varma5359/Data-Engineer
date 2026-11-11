
# Day 35 Project Workbook — TechStore Data Cleaning Framework

**Goal:** Profile partner order data, apply approved cleaning rules, preserve rejected records with reasons, detect schema drift and reconcile the result.  
**Time:** 90–120 minutes. **Prerequisites:** Pandas, Day 16 validation and Day 34 transformation.

## 1. Business scenario and rules

TechStore receives order files from a partner. The file may contain blank IDs, duplicate order IDs, invalid amounts, inconsistent status spelling and invalid dates. We must not silently delete suspicious records.

Agreed rules:

| Field | Rule | Failure action |
|---|---|---|
| order_id | Required, trimmed, unique | Reject |
| customer_id | Required | Reject |
| amount | Numeric and >= 0 | Reject |
| status | PAID, PENDING or CANCELLED | Reject |
| order_date | Valid date | Reject |

We will reject every row sharing a duplicate business key. Do not arbitrarily keep the first duplicate unless the business owner defines a winner rule.

## 2. Create workspace and install

CMD:
~~~bat
mkdir day35_cleaning_lab
cd day35_cleaning_lab
mkdir data reports
python -m pip install pandas
~~~

Create a Jupyter notebook named day35_cleaning.ipynb. Run cells in order.

## 3. Create reproducible source data

~~~python
import pandas as pd
from pathlib import Path

data_dir = Path("data")
report_dir = Path("reports")
data_dir.mkdir(exist_ok=True)
report_dir.mkdir(exist_ok=True)

raw = pd.DataFrame({
    "order_id": [" O1001 ", "O1002", "O1002", "O1004", "", "O1006"],
    "customer_id": ["C01", "C02", "C02", "C04", "C05", "C06"],
    "amount": ["120.50", "bad-value", "bad-value", "0", "55.00", "-10"],
    "status": [" paid ", "PENDING", "PENDING", "paid", "cancelled", "unknown"],
    "order_date": ["2026-01-01", "2026-01-02", "2026-01-02", "not-a-date", "2026-01-05", "2026-01-06"]
})
raw_path = data_dir / "orders_raw.csv"
raw.to_csv(raw_path, index=False)
print("Created:", raw_path)
print("Input rows:", len(raw))
~~~

Expected: file path data/orders_raw.csv and 6 input rows. Keep this raw file unchanged.

## 4. Load and profile before changing anything

~~~python
source = pd.read_csv(raw_path, dtype="string")
print("Shape:", source.shape)
print("Columns:", source.columns.tolist())
print(source.isna().sum())
print("Statuses:", source["status"].unique().tolist())
~~~

Expected shape: (6, 5). Profiling tells us what arrived. It is not cleaning. Values such as amount remain text at this point.

## 5. Normalize approved formatting and parse types

~~~python
clean = source.copy()
clean["order_id"] = clean["order_id"].str.strip()
clean["customer_id"] = clean["customer_id"].str.strip()
clean["status"] = clean["status"].str.strip().str.upper()
clean["amount"] = pd.to_numeric(clean["amount"], errors="coerce")
clean["order_date"] = pd.to_datetime(clean["order_date"], errors="coerce")
print(clean[["order_id", "amount", "status", "order_date"]].to_string(index=False))
~~~

Explanation:
- copy protects the loaded source.
- strip removes surrounding spaces.
- upper standardizes status casing.
- to_numeric converts unparseable text to NaN; it does not make that value valid.
- to_datetime converts invalid dates to NaT.

## 6. Collect all rejection reasons

~~~python
allowed_statuses = ["PAID", "PENDING", "CANCELLED"]
duplicate_key = (
    clean["order_id"].notna()
    & clean["order_id"].ne("")
    & clean["order_id"].duplicated(keep=False)
)

reasons = pd.Series("", index=clean.index, dtype="string")
checks = [
    (clean["order_id"].isna() | clean["order_id"].eq(""), "missing_order_id"),
    (clean["customer_id"].isna() | clean["customer_id"].eq(""), "missing_customer_id"),
    (duplicate_key, "duplicate_order_id"),
    (clean["amount"].isna(), "invalid_amount"),
    (clean["amount"].lt(0), "negative_amount"),
    (~clean["status"].isin(allowed_statuses), "unknown_status"),
    (clean["order_date"].isna(), "invalid_order_date")
]
for condition, reason in checks:
    reasons = reasons.where(~condition, reasons + reason + ";")

clean["rejection_reasons"] = reasons.str.rstrip(";")
accepted = clean[clean["rejection_reasons"].eq("")].copy()
rejected = clean[~clean["rejection_reasons"].eq("")].copy()
print("Accepted:", len(accepted))
print("Rejected:", len(rejected))
print(rejected[["order_id", "rejection_reasons"]].to_string(index=False))
~~~

Expected for this exact fixture: accepted = 1 (O1001), rejected = 5. Both O1002 rows are rejected as duplicate keys and invalid amounts. O1004 has an invalid date. The blank ID is rejected. O1006 has a negative amount and unknown status. One row may have multiple reasons.

## 7. Save outputs and summary

~~~python
accepted.to_csv(data_dir / "orders_accepted.csv", index=False)
rejected.to_csv(data_dir / "orders_rejected.csv", index=False)

summary = pd.DataFrame([
    {"metric": "input_rows", "value": len(source)},
    {"metric": "accepted_rows", "value": len(accepted)},
    {"metric": "rejected_rows", "value": len(rejected)}
])
summary.to_csv(report_dir / "cleaning_summary.csv", index=False)
print(summary.to_string(index=False))
~~~

Expected summary: input_rows 6, accepted_rows 1, rejected_rows 5.

## 8. Reconcile and prove accepted data satisfies the contract

~~~python
assert len(source) == len(accepted) + len(rejected)
assert accepted["order_id"].is_unique
assert accepted["order_id"].notna().all()
assert accepted["amount"].ge(0).all()
assert accepted["status"].isin(allowed_statuses).all()
assert accepted["order_date"].notna().all()
print("Reconciliation passed")
~~~

Expected: Reconciliation passed. If an assertion fails, stop publication and investigate.

## 9. Detect schema drift

~~~python
expected = {"order_id", "customer_id", "amount", "status", "order_date"}
received = {"order_id", "customer_id", "order_total", "status", "order_date", "sales_channel"}
print("Missing:", sorted(expected - received))
print("Extra:", sorted(received - expected))
~~~

Expected missing: amount. Expected extra: order_total and sales_channel. A possible rename is not proof that two fields have the same meaning. Confirm currency, tax treatment, precision and business definition with the source owner.

An approved mapping can be applied explicitly:

~~~python
approved_mapping = {"order_total": "amount"}
incoming = pd.DataFrame({
    "order_id": ["O2001"], "customer_id": ["C20"],
    "order_total": ["88.00"], "status": ["PAID"],
    "order_date": ["2026-01-10"], "sales_channel": ["web"]
})
mapped = incoming.rename(columns=approved_mapping)
print(mapped.columns.tolist())
~~~

This mapping is approved only for this exercise. In a real pipeline, version and document the contract change.

## 10. Failure drills

1. Change an amount to text. Confirm invalid_amount is added.
2. Add a second O1001. Confirm both rows are rejected.
3. Remove customer_id. Stop before processing because a required column is missing.
4. Add optional sales_channel. Decide whether downstream consumers tolerate it.
5. Run twice. Confirm raw data is unchanged and output is reproducible.

## 11. Troubleshooting

| Symptom | Cause | Correction |
|---|---|---|
| KeyError for a field | Required column absent or misspelled | Compare schema before accessing fields |
| NaN after conversion | Text cannot be parsed as a number | Inspect source value; do not replace blindly with zero |
| Duplicate data disappears | drop_duplicates used without a business rule | Quarantine duplicate keys or apply an approved winner rule |
| Accepted + rejected is less than input | Rows were dropped before splitting | Preserve original rows and reconcile |
| Date becomes NaT | Invalid or unexpected date format | Inspect source format and timezone contract |

## 12. Submission and acceptance checklist

Submit raw input, accepted output, rejected output with reasons, cleaning summary, schema comparison and notebook output.

- [ ] Raw input unchanged.
- [ ] Profiling performed before cleaning.
- [ ] Required fields checked.
- [ ] Duplicate business keys handled deliberately.
- [ ] Invalid numbers and dates detected.
- [ ] Every rejected row has a reason.
- [ ] Schema drift reported, not guessed away.
- [ ] Input = accepted + rejected.
- [ ] Accepted rows satisfy every rule.
- [ ] Rerun produces the same result.

**Explain your project:** "I profiled partner order data, applied documented validation and normalization rules, quarantined invalid rows with reasons, compared the incoming schema with the contract and reconciled every input row before publishing accepted data."
