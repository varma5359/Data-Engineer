# Day 6 — Python Environment, Syntax, Types and Data Structures
**Phase 2: Python for Data Engineering | Beginner**

## Outcomes
Set up Python, run scripts/notebooks, understand variables and core types, and select a data structure for a small processing task.

## 1. Python in Data Engineering
Python is widely used for ingestion, API calls, file processing, transformations, validation, automation, and orchestration support. SQL remains important for database-side work; Python coordinates and handles logic around it.

## 2. Environment
- Install a supported Python version; verify with `python --version` or `py --version`.
- Use a virtual environment per project to isolate dependencies.
- VS Code is an editor; Jupyter is an interactive notebook; Colab is hosted notebook execution.
- A `.py` file runs as a script; a notebook runs cells progressively.

Windows CMD:
```bash
mkdir data_processing_utility
cd data_processing_utility
py -m venv .venv
.venv\Scripts\activate
python --version
python -m pip install pandas
```
Keep a dependency list (`requirements.txt`) so another developer can reproduce the environment.

## 3. Variables and types
```python
source_name = "sales_api"       # str
record_count = 1250              # int
load_success = True              # bool
amount = 2499.50                 # float (money usually Decimal in financial work)
missing_value = None             # no value
print(source_name, record_count, load_success, amount, missing_value)
```
Output: `sales_api 1250 True 2499.5 None`

Use meaningful names. Python is dynamically typed, but data pipelines still need explicit validation and predictable types.

## 4. Collections
```python
columns = ["order_id", "amount", "status"]  # list: ordered, mutable
source = ("orders", "postgres")              # tuple: ordered, immutable
required = {"order_id", "amount"}            # set: unique values
config = {"source": "orders", "batch_size": 500}  # dict: key/value
print(columns[0], config["batch_size"], "amount" in required)
```
Output: `order_id 500 True`

- List: sequence of records/columns.
- Tuple: fixed grouping.
- Set: uniqueness/membership.
- Dictionary: record/config mapping.

## 5. Simple record processing
```python
orders = [
    {"order_id": 101, "amount": 500},
    {"order_id": 102, "amount": 750}
]
total = 0
for order in orders:
    total = total + order["amount"]
print("Orders:", len(orders))
print("Revenue:", total)
```
Output: `Orders: 2` and `Revenue: 1250`.

## 6. Mini-project — Data Processing Utility
Build a small script that stores source configuration, counts records, calculates a total, and prints a summary. Keep input data separate from processing code.

## Common errors
- Using `=` instead of `==` in comparisons.
- Accessing a missing dictionary key.
- Treating numeric-looking IDs as numbers.
- Installing packages outside the active environment.
- Using float for exact financial arithmetic without considering precision.

## Revision
Environment isolates dependencies; variables name values; list/tuple/set/dict serve different purposes; type correctness must be checked at data boundaries.