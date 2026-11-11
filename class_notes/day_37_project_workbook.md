# Day 37 Project Workbook — Incremental Data Loader

## Goal
Load only new/changed source records, update existing target keys, and advance a watermark only after target verification.

## 1. Create source fixture
```python
import pandas as pd
source = pd.DataFrame({
 "order_id":[101,102,103,102],
 "customer_id":["C1","C2","C3","C2"],
 "amount":[100,200,150,225],
 "updated_at":["2026-01-01 09:00","2026-01-01 10:00","2026-01-01 11:00","2026-01-02 08:00"]})
source["updated_at"] = pd.to_datetime(source["updated_at"])
target = pd.DataFrame(columns=source.columns)
watermark = pd.Timestamp("1900-01-01")
```
The final row updates order 102. The unique business key is order_id; updated_at determines which version is later in this fixture.

## 2. Select changed rows
```python
changes = source[source["updated_at"] > watermark].copy()
print(len(changes))
```
Expected: 4.

## 3. Select latest version per key
```python
latest = changes.sort_values("updated_at").drop_duplicates("order_id", keep="last")
print(latest[["order_id","amount"]].sort_values("order_id"))
```
Expected: 101→100, 102→225, 103→150.

## 4. Upsert teaching simulation
```python
combined = pd.concat([target, latest], ignore_index=True)
target = combined.sort_values("updated_at").drop_duplicates("order_id", keep="last")
target = target.sort_values("order_id").reset_index(drop=True)
print(target[["order_id","amount"]])
```
Target contains three unique order IDs. In a database, use a unique constraint and transactional INSERT ... ON CONFLICT / MERGE syntax appropriate to the database.

## 5. Advance only after verification
```python
assert target["order_id"].is_unique
assert set(latest["order_id"]).issubset(set(target["order_id"]))
new_watermark = changes["updated_at"].max()
print("Rows:", len(target), "Watermark:", new_watermark)
```
Expected rows 3; watermark 2026-01-02 08:00. A production checkpoint must be durable and coordinated with target commit.

## 6. Rerun and boundary challenge
Repeat selection using the saved watermark: no later records should be selected in this static fixture. Discuss why strict greater-than can miss equal timestamps and why composite cursor/overlap plus deduplication may be needed. Add a late-arriving older timestamp and propose a lookback/reconciliation policy.

## Acceptance
One row per key; order 102 has latest amount; no checkpoint before target success; explain tie/late-arrival behavior; reconcile target keys and counts.
