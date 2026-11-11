# Day 35 — Data Cleaning and Schema Evolution

**Phase:** 4 — Data Processing and Pipelines  
**Syllabus topic:** Data Cleaning and Schema Evolution  
**Mini-project:** TechStore Data Cleaning Framework  
**Audience:** Complete beginner / fresher  
**Study time:** 2–3 hours (pause and run every cell)

> **How to use this lesson:** Read one section, run its code in a notebook, compare your output, and explain it aloud. Do not skip directly to the final project. The project is assembled from the same small steps.

## 0. Today's learning contract

At the end, you should be able to:
- explain dirty data, profiling, validation, cleaning, schema, schema drift and schema evolution in plain English;
- distinguish a missing value from an invalid value and a duplicate;
- choose a cleaning rule based on business meaning rather than guessing;
- detect missing, extra and renamed columns;
- classify schema changes as additive, compatible, breaking or unknown;
- preserve rejected records and reasons;
- build and test a small cleaning pipeline;
- explain what the pipeline changed and prove that no records disappeared silently.

**You are not expected to memorize every pandas method today.** Understand the decisions first; methods are tools that implement those decisions.

## 1. The business problem: why cleaning exists

Imagine TechStore receives an orders file from a partner every morning. Yesterday's file contained:

| order_id | customer_id | amount | status |
|---|---|---:|---|
| O101 | C10 | 1200 | paid |
| O102 | C11 | 800 | paid |

Today's file contains:

| order_id | customerId | amount | status |
|---|---|---|---|
| O103 | C12 | 500 | PAID |
| O104 | *(blank)* | 700 | paid |
| O104 | *(blank)* | 700 | paid |
| O105 | C13 | -90 | refund |

What changed?
- The partner renamed `customer_id` to `customerId`.
- A customer identifier is missing.
- Order O104 appears twice.
- Status casing is inconsistent (`PAID` vs `paid`).
- A negative amount appears. It may be invalid—or it may represent a refund. Business meaning must be checked.

**Mentor point:** Cleaning is not “make every value look nice.” It is a controlled decision about which values can be corrected, accepted, rejected or investigated.

## 2. Vocabulary from zero

| Term | Simple meaning | TechStore example |
|---|---|---|
| Raw data | Data as received, unchanged | Partner's original CSV |
| Dirty data | Data that violates a rule or is inconsistent | Blank required customer ID |
| Profiling | Inspecting data before changing it | Count nulls and duplicate IDs |
| Validation | Checking data against rules | Amount must be numeric |
| Cleaning | Applying approved correction/removal rules | Trim spaces from status |
| Schema | Column names, types and structure | order_id, amount, status |
| Schema drift | Unexpected schema difference | customer_id suddenly becomes customerId |
| Schema evolution | A controlled, approved schema change | Add optional currency column |
| Data contract | Agreement about structure, meaning and quality | order_id is required and unique |
| Quarantine | Keep problematic data separately for review | Row with missing customer ID |
| Idempotent | Repeating the same run does not create extra effects | Re-run does not duplicate cleaned orders |

## 3. The safe cleaning workflow

```text
Receive raw data
      ↓
Preserve original (never overwrite it)
      ↓
Profile data (shape, types, nulls, duplicates, ranges)
      ↓
Check schema / contract
      ↓
Apply approved cleaning rules
      ↓
Validate cleaned data again
      ↓
Accepted records + rejected records + reasons
      ↓
Reconcile counts and publish
```

A beginner mistake is to clean immediately. First inspect what arrived. If you change values before profiling, you may destroy evidence of the source problem.

## 4. Set up today's notebook

Use the Python environment from Phase 2. In a notebook, execute each code cell in order.

```python
import pandas as pd
```

**What this line does:** `import` loads a library; `pandas` is the library used for tabular data; `pd` is the conventional short alias. If you see `ModuleNotFoundError: No module named 'pandas'`, install pandas in the same environment/kernel and restart if needed.

## 5. Create a tiny dataset (understand every row first)

```python
orders = pd.DataFrame({
    "order_id": ["O101", "O102", "O103", "O104", "O104", "O105"],
    "customer_id": ["C10", "C11", "C12", None, None, "C13"],
    "amount": ["1200", "800", "500", "700", "700", "-90"],
    "status": [" paid ", "paid", "PAID", "paid", "paid", "refund"]
})
orders
```

Expected table:

| index | order_id | customer_id | amount | status |
|---:|---|---|---|---|
| 0 | O101 | C10 | 1200 | ' paid ' |
| 1 | O102 | C11 | 800 | paid |
| 2 | O103 | C12 | 500 | PAID |
| 3 | O104 | None | 700 | paid |
| 4 | O104 | None | 700 | paid |
| 5 | O105 | C13 | -90 | refund |

**Read it carefully:** the amount column contains text strings, not numeric values. The two O104 rows share the same order ID. `None` represents a missing value in this sample.

### 5.1 What is a DataFrame?

A DataFrame is a table-like object with rows and columns. The `orders` variable points to the table we created. The index (0–5) is a row label—not the order ID and not a business key.

## 6. Profile before cleaning

### 6.1 Shape: how many rows and columns?

```python
orders.shape
```

Expected output:

```text
(6, 4)
```

Meaning: 6 rows and 4 columns. This is a baseline; later we compare it with accepted/rejected counts.

### 6.2 Column names

```python
orders.columns
```

Expected output:

```text
Index(['order_id', 'customer_id', 'amount', 'status'], dtype='object')
```

This tells us the exact labels currently available. Column names are case-sensitive.

### 6.3 Data types

```python
orders.dtypes
```

Expected output (format may vary slightly by pandas version):

```text
order_id       object
customer_id    object
amount         object
status         object
dtype: object
```

Why is amount `object`? We supplied strings such as `"1200"`. A string can contain non-numeric text, so pandas cannot safely assume it is a number.

### 6.4 Missing values per column

```python
orders.isna().sum()
```

Expected output:

```text
order_id       0
customer_id    2
amount         0
status         0
dtype: int64
```

How to read it:
- `isna()` checks each cell and returns True for missing values.
- `sum()` counts True values column-wise.
- Two customer IDs are missing. This is a finding—not permission to invent customer IDs.

### 6.5 Duplicate order IDs

```python
orders["order_id"].duplicated().sum()
```

Expected output:

```text
1
```

The first O104 is treated as the original occurrence; the second is marked duplicated. This method only detects repeated values in the chosen column. It does not decide which record is correct.

### 6.6 Inspect unique status values

```python
orders["status"].value_counts(dropna=False)
```

Expected counts:

```text
paid       3
PAID       1
refund     1
 paid      1
```

We have three visual forms of paid. We can standardize case and surrounding spaces if the business contract says status values are case-insensitive.

## 7. Cleaning rule 1 — Trim whitespace

### Why?
A value like `" paid "` looks like paid to a person, but exact comparisons may fail because spaces are part of the string.

### Code

```python
orders["status"] = orders["status"].str.strip()
orders["status"]
```

Expected values:

```text
0      paid
1      paid
2      PAID
3      paid
4      paid
5    refund
Name: status, dtype: object
```

**Breakdown**
- `orders["status"]`: select the status column.
- `.str`: access string operations for each value.
- `.strip()`: remove whitespace at the beginning and end.
- Assignment writes the cleaned values back into this working DataFrame.

**Important:** this changes the working copy. In a production pipeline, retain the raw input separately so you can trace what the source sent.

## 8. Cleaning rule 2 — Standardize case

```python
orders["status"] = orders["status"].str.lower()
orders["status"].value_counts()
```

Expected output:

```text
paid      5
refund    1
Name: count, dtype: int64
```

Now `PAID` becomes `paid`. We used lower case because the example contract defines canonical status values in lowercase.

**Do not lowercase every field automatically.** Names, codes and case-sensitive identifiers may require preserving case.

## 9. Cleaning rule 3 — Convert amount safely

### The problem
A text value `"1200"` cannot be reliably summed as a number until it is converted. A bad value such as `"unknown"` should not crash the whole batch without an intentional policy.

### First: safe conversion

```python
orders["amount_numeric"] = pd.to_numeric(orders["amount"], errors="coerce")
orders[["amount", "amount_numeric"]]
```

Expected result:

| amount | amount_numeric |
|---|---:|
| 1200 | 1200 |
| 800 | 800 |
| 500 | 500 |
| 700 | 700 |
| 700 | 700 |
| -90 | -90 |

`errors="coerce"` converts values that cannot be parsed into missing values (NaN). It does not mean the record is valid; we must detect conversion failures next.

### Detect conversion failures

```python
bad_amount = orders["amount_numeric"].isna()
orders.loc[bad_amount, ["order_id", "amount"]]
```

If a source amount were `"unknown"`, this would show its order ID and original value. In today's six rows there are no conversion failures.

### Negative amount: don't guess

A negative amount might mean:
- invalid order amount;
- refund/credit;
- reversal or adjustment.

Ask the business owner or inspect the source transaction type. For this lesson, our contract says **order amount must be zero or positive; refunds are represented in a separate refund feed**. Under that stated contract, -90 is rejected. In another system, the rule may differ.

## 10. Cleaning rule 4 — Required fields

A required field cannot be missing for a valid output record.

```python
missing_customer = orders["customer_id"].isna()
orders.loc[missing_customer, ["order_id", "customer_id"]]
```

Expected output: rows O104 and O104 with missing customer ID.

**What should we do?**
- Do not fill with `"UNKNOWN"` unless the downstream contract explicitly allows that value.
- Do not assign a random customer ID.
- Keep the original record in rejected/quarantine output and record reason `missing_customer_id`.

## 11. Cleaning rule 5 — Duplicate identity

A duplicate is defined using a **business key**, not merely “all columns look alike.”

For this order feed, the source contract says `order_id` uniquely identifies an order. Therefore repeated order IDs violate the contract.

```python
duplicate_order = orders["order_id"].duplicated(keep="first")
orders.loc[duplicate_order]
```

Expected output: the second O104 row.

`keep="first"` marks later occurrences as duplicates. This is only a detection demonstration. A production decision must consider:
- Is the source file a full snapshot or a change feed?
- Does the same order ID represent updates?
- Is there an event ID or version?
- Which record is authoritative (latest update, highest source version, etc.)?
- Can legitimate multiple events share the same order ID?

**Never delete duplicates blindly.** Deduplication policy must follow source semantics.

## 12. Schema checks from zero

A schema is the agreed structure of a dataset: names, types, required/optional status and sometimes constraints/meaning.

### 12.1 Expected vs received columns

```python
expected_columns = {"order_id", "customer_id", "amount", "status"}
received_columns = set(orders.columns)

missing_columns = expected_columns - received_columns
extra_columns = received_columns - expected_columns

print("Missing:", missing_columns)
print("Extra:", extra_columns)
```

Expected:

```text
Missing: set()
Extra: set()
```

### 12.2 Simulate supplier rename

```python
partner_today = orders.rename(columns={"customer_id": "customerId"})
received_columns = set(partner_today.columns)

print("Missing:", expected_columns - received_columns)
print("Extra:", received_columns - expected_columns)
```

Expected:

```text
Missing: {'customer_id'}
Extra: {'customerId'}
```

**Interpretation:** a rename appears as one missing expected column plus one unexpected column. Do not silently assume the new name is correct.

### 12.3 Approved rename mapping

If the supplier confirms that `customerId` has exactly the same meaning and semantics as `customer_id`, document and apply a mapping:

```python
partner_today = partner_today.rename(columns={"customerId": "customer_id"})
```

Then run the schema check again. A mapping is not just code: it is an approved contract change with owner, effective date and tests.

## 13. Schema evolution: classify before accepting

| Change | Example | Typical impact | Safe response |
|---|---|---|---|
| Add optional field | add `currency` | Often compatible | Verify consumers ignore unknown fields or support it |
| Add required field | new mandatory `country` | May break old producers | Version contract / coordinated rollout |
| Rename field | `customer_id` → `customerId` | Usually breaking | Explicit mapping or new contract version |
| Remove field | delete `status` | Breaking for consumers | Deprecate, migrate, then remove |
| Type change | integer → string | Potentially breaking | Validate conversion and downstream impact |
| Meaning change | amount now includes tax | Semantically breaking even if type unchanged | New definition/version and consumer review |
| Reorder columns | same names, different order | Depends on positional consumers | Prefer name-based access; test positional systems |

### Backward compatibility
A new producer version is backward-compatible when existing consumers can continue to work under the agreed contract. Adding an optional field is often compatible for name-based consumers, but not universally—for example, strict schema validators may reject unknown fields.

### Forward compatibility
An older consumer can handle data produced by a newer schema. This also depends on consumer design and contract rules; it is not guaranteed by adding a column.

## 14. Build a validation report (small, understandable)

```python
report = {
    "input_rows": len(orders),
    "missing_customer_id": int(orders["customer_id"].isna().sum()),
    "duplicate_order_id_rows": int(orders["order_id"].duplicated().sum()),
    "amount_conversion_failures": int(orders["amount_numeric"].isna().sum()),
    "negative_amount_rows": int((orders["amount_numeric"] < 0).sum())
}
report
```

Expected:

```text
{'input_rows': 6, 'missing_customer_id': 2,
 'duplicate_order_id_rows': 1, 'amount_conversion_failures': 0,
 'negative_amount_rows': 1}
```

This is a **profile report**, not yet a final accepted/rejected report. A profile tells us what is present; a validation decision applies the rules.

## 15. Mini-project: TechStore Data Cleaning Framework

### Business requirement
Partner order files must be cleaned before the analytics team uses them. We must:
1. preserve raw records;
2. normalize status whitespace/case;
3. convert amount to numeric;
4. require order ID and customer ID;
5. reject negative amount under today's contract;
6. detect duplicate order IDs;
7. output accepted and rejected records with reasons;
8. reconcile input = accepted + rejected.

### 15.1 Start from a fresh raw dataset

```python
raw_orders = pd.DataFrame({
    "order_id": ["O201", "O202", "O203", "O204", "O204", "O205", "O206"],
    "customer_id": ["C21", "C22", "C23", None, None, "C25", "C26"],
    "amount": ["1500", "900", "500", "700", "700", "-90", "bad-value"],
    "status": [" PAID ", "paid", "PAID", "paid", "paid", "refund", "pending"]
})
```

The raw dataset has seven records. Keep `raw_orders` unchanged as evidence.

### 15.2 Create a working copy

```python
clean_orders = raw_orders.copy()
```

Why `.copy()`? It gives us a separate DataFrame to transform. We can compare it with the original later.

### 15.3 Normalize status and convert amount

```python
clean_orders["status"] = clean_orders["status"].str.strip().str.lower()
clean_orders["amount_numeric"] = pd.to_numeric(clean_orders["amount"], errors="coerce")
```

The chained string operations mean: strip outer whitespace first, then convert to lowercase.

### 15.4 Build one reason per row (simple beginner approach)

```python
clean_orders["reject_reason"] = ""

clean_orders.loc[clean_orders["order_id"].isna(), "reject_reason"] = "missing_order_id"
clean_orders.loc[clean_orders["customer_id"].isna(), "reject_reason"] = "missing_customer_id"
clean_orders.loc[clean_orders["amount_numeric"].isna(), "reject_reason"] = "invalid_amount"
clean_orders.loc[clean_orders["amount_numeric"] < 0, "reject_reason"] = "negative_amount"
```

This simple version stores one reason. If a record violates multiple rules, later assignments overwrite earlier reasons. A production-quality framework should collect **all** reasons; the workbook demonstrates that improvement.

### 15.5 Mark duplicates

```python
duplicate_mask = clean_orders["order_id"].duplicated(keep="first")
clean_orders.loc[duplicate_mask, "reject_reason"] = "duplicate_order_id"
```

For this practice dataset, the second O204 is marked duplicate. If it already had another reason, this assignment would overwrite it; that is a known limitation we fix in the workbook's advanced exercise.

### 15.6 Split accepted and rejected

```python
accepted = clean_orders[clean_orders["reject_reason"] == ""].copy()
rejected = clean_orders[clean_orders["reject_reason"] != ""].copy()

print("Input:", len(raw_orders))
print("Accepted:", len(accepted))
print("Rejected:", len(rejected))
print("Reconciled:", len(accepted) + len(rejected) == len(raw_orders))
```

Expected output for the stated rules and sample:

```text
Input: 7
Accepted: 3
Rejected: 4
Reconciled: True
```

Accepted orders: O201, O202, O203.  
Rejected rows: missing customer (O204 first), duplicate O204, negative O205, invalid amount O206.

### 15.7 Inspect rejected records

```python
rejected[["order_id", "customer_id", "amount", "reject_reason"]]
```

Expected reason categories:
- O204 — missing_customer_id
- O204 — duplicate_order_id
- O205 — negative_amount
- O206 — invalid_amount

The rejected dataset is not trash. It is evidence for the source owner, audit, correction and controlled replay.

### 15.8 Save outputs

```python
accepted.to_csv("accepted_orders.csv", index=False)
rejected.to_csv("rejected_orders.csv", index=False)
```

- `to_csv` writes a CSV file.
- `index=False` prevents pandas' row index from becoming an extra business column.
- The files are created in the notebook's current working directory. Use a known project output folder in a real project.

### 15.9 Verify files

```python
accepted_check = pd.read_csv("accepted_orders.csv")
rejected_check = pd.read_csv("rejected_orders.csv")

print("Accepted file rows:", len(accepted_check))
print("Rejected file rows:", len(rejected_check))
```

Expected: 3 and 4. Reading the files back verifies that the outputs can be reopened; it does not prove every business rule is correct. Rule-level tests are also required.

## 16. Common beginner errors and fixes

| Mistake | Why it happens | Better approach |
|---|---|---|
| Replace every missing value with zero | Zero looks convenient | Decide field meaning; zero and unknown are different |
| Drop duplicates on all columns | Confuses identical payload with business identity | Define event/order key and source semantics |
| Rename columns silently | Assumes source change is harmless | Contract approval, mapping, tests and version |
| Convert amount without checking failures | Invalid strings become NaN or raise errors | Count conversion failures and route them |
| Remove negative amounts automatically | Assumes negatives are always wrong | Confirm refunds/reversals business meaning |
| Overwrite raw file | Loses source evidence | Immutable raw layer and separate working copy |
| Reject whole file for one bad row | No record-level policy | Decide batch-fatal vs row-reject rules |
| Accept whole file despite missing required column | Code may continue with incomplete data | Schema check before processing |
| Ignore rejected rows | Accepted output looks clean | Reconcile and publish reject reasons/counts |
| Mark run successful before output is durable | Checkpoint moves too early | Persist, verify, then checkpoint |

## 17. Debugging drills

### Drill A: `KeyError: 'customer_id'`
**Cause:** incoming file has `customerId`, not `customer_id`.  
**Check:** print `df.columns`; compare received vs expected schema.  
**Fix:** use an approved mapping; do not guess.

### Drill B: `ValueError` while converting amount
**Cause:** string contains non-numeric text.  
**Fix:** use controlled conversion, capture failures, and reject/quarantine with original value.

### Drill C: Output row count is smaller
**Cause:** filtering, dropping duplicates, null removal or failed writes.  
**Fix:** compare raw count, accepted count, rejected count and output count. Explain every difference.

### Drill D: Rerun creates duplicate outputs
**Cause:** append behavior without idempotency or output cleanup/versioning.  
**Fix:** define run ID, output overwrite/merge policy, unique key and safe replay.

## 18. Data quality dimensions

| Dimension | Question | Example check |
|---|---|---|
| Completeness | Is required data present? | customer_id not null |
| Uniqueness | Is identity unique? | order_id unique |
| Validity | Does value follow allowed rules? | amount numeric and non-negative |
| Consistency | Do related fields agree? | status and refund type align |
| Accuracy | Does value reflect reality? | reconcile with source transaction |
| Timeliness | Is data arriving when expected? | file received by deadline |

A dataset can pass validity and still be inaccurate. A numeric amount may be the wrong amount. Quality checks are evidence, not a guarantee of truth.

## 19. Production engineering considerations

- **Raw retention:** keep original file/object and source metadata according to retention policy.
- **Contract version:** record contract/schema version with each run.
- **Observability:** record input/accepted/rejected counts, duration, failures and source file name.
- **Security:** avoid exposing personal data in logs; restrict raw/rejected access.
- **Scale:** profile bounded batches; avoid loading huge files entirely into memory.
- **Idempotency:** reprocessing the same source should not duplicate target records.
- **Ownership:** define who approves schema changes and who investigates rejects.
- **Recovery:** keep enough metadata to replay corrected records safely.

## 20. Exercises (attempt before reading workbook)

1. Add a row with blank order ID. What should happen?
2. Add amount `"1,200"`. Does `to_numeric` parse it directly? Test and design a locale-aware rule.
3. Add a status `" Paid "`. Which normalization steps are needed?
4. Add a new optional column `currency`. Is the change automatically safe?
5. Rename `amount` to `total_amount`. What checks should fail?
6. Create two rows with the same order ID but different amounts. Why is “keep first” insufficient?
7. Make one row violate two rules. How can you retain both reasons?
8. Prove `input = accepted + rejected` after every run.

## 21. Day 35 revision checklist

You are ready to move on when you can explain without looking:
- [ ] profiling vs validation vs cleaning;
- [ ] why raw data is preserved;
- [ ] why a missing value is not automatically zero;
- [ ] how to detect duplicate business keys;
- [ ] how to detect schema drift;
- [ ] why a rename/type/meaning change may break consumers;
- [ ] how accepted and rejected rows are reconciled;
- [ ] how to rerun safely and investigate failures.

## 22. What comes next?

Day 36 builds on cleaned data and teaches **Batch Processing and Partitioning**: how to process bounded chunks, divide large datasets into partitions, choose partition keys, handle skew and validate partition-level results.

---

**Student reminder:** If you cannot explain why a record was accepted or rejected, the pipeline is not finished.