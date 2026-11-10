# Day 6 Interview Preparation: Python Introduction and Environment

**Mini-project:** Data Processing Utility

## 1. Core questions

1. **Why is Python used in data engineering?**  
   It reads and writes files, connects to APIs and databases, applies business rules and automates pipeline steps. SQL remains important for relational data operations.

2. **What is a Python interpreter?**  
   It executes Python source code. The interpreter version and environment determine which packages are available.

3. **What is the difference between a script and a notebook?**  
   A script is a saved program commonly stored in a `.py` file. A notebook combines code cells, outputs and explanatory text for interactive exploration.

4. **What is a variable?**  
   A name that refers to a value.

5. **What does dynamically typed mean?**  
   Python determines the type of a value at runtime. A variable name can later refer to a value of another type.

6. **What are common built-in data types?**  
   `str`, `int`, `float`, `bool`, `list`, `tuple`, `set`, `dict` and `NoneType`.

7. **List vs tuple?**  
   Both are ordered. A list is mutable; a tuple is immutable.

8. **List vs set?**  
   A list preserves order and can contain duplicates. A set stores unique elements and is useful for membership and uniqueness checks.

9. **What is a dictionary?**  
   A key-value collection. It is useful for representing one record with named fields.

10. **What does `type()` do?**  
    It reports the type of a value.

11. **What is type conversion?**  
    Converting a value to another type, such as `int("5")` or `float("12.5")`.

12. **What happens when `int("five")` runs?**  
    Python raises `ValueError` because the text is not a valid integer representation.

13. **Why use meaningful variable names?**  
    They make business logic easier to read, review and maintain.

14. **What is a virtual environment?**  
    An isolated Python environment for a project's dependencies.

15. **Why run `python -m pip install package`?**  
    It runs pip through the selected Python interpreter and reduces confusion between multiple installations.

## 2. Difference questions

| Question | Strong answer |
|---|---|
| Python vs SQL | Python handles general logic and integrations; SQL is designed for relational data querying and manipulation |
| Script vs notebook | Script is a reusable program; notebook supports interactive cells and exploration |
| List vs tuple | List is mutable; tuple is immutable |
| Set vs dictionary | Set stores unique values; dictionary maps keys to values |
| Integer vs float | Integer represents whole numbers; float represents approximate decimal values |
| `print` vs `return` | Print displays; return sends a value back to the caller |
| CMD vs interpreter | CMD runs operating-system commands; Python interpreter executes Python code |

## 3. Practical coding questions

**Q1. Convert a quantity from text to integer.**

```python
quantity_text = "4"
quantity = int(quantity_text)
print(quantity)
```

Output: `4`.

**Q2. Calculate line amount.**

```python
quantity = 3
unit_price = 250
print(quantity * unit_price)
```

Output: `750`.

**Q3. Access an order field.**

```python
order = {"order_id": 501, "status": "PAID"}
print(order["status"])
```

Output: `PAID`.

**Q4. Find unique statuses.**

```python
statuses = ["PAID", "PENDING", "PAID"]
print(sorted(set(statuses)))
```

Output: `['PAID', 'PENDING']`.

**Q5. Calculate total sales from records.**

```python
orders = [{"amount": 100}, {"amount": 250}]
total = 0
for order in orders:
    total += order["amount"]
print(total)
```

Output: `350`.

## 4. Scenario-based questions

1. **A CSV quantity arrives as text. What do you do?**  
   Inspect the raw value, convert to integer, handle invalid values and retain the rejection reason. Do not silently turn invalid quantities into zero.

2. **A package imports in CMD but not in Jupyter. Why?**  
   CMD and Jupyter may use different Python interpreters. Compare interpreter paths and install the package into the notebook's environment.

3. **You need to store one order with named fields. Which structure?**  
   A dictionary, because keys such as `order_id` and `amount` make the fields explicit.

4. **You need to store many order records. Which structure?**  
   A list of dictionaries is a simple in-memory representation for a small dataset. For large data, use file, database or dataframe processing.

5. **A total is unexpectedly concatenated as text. What happened?**  
   The source value may be a string. Inspect types and convert numeric fields before calculation.

## 5. Follow-up questions interviewers may ask

- What does zero-based indexing mean?
- What is the difference between `append()` and `extend()`?
- Why can dictionary key access raise `KeyError`?
- What does `None` represent?
- Why should secrets not be stored in source code?
- What is the purpose of `requirements.txt`?
- How do you check which Python executable is running?

## 6. Strong revision points

- Python and SQL solve complementary problems.
- Understand collection types and their behavior, not only their syntax.
- Convert source values deliberately and validate them.
- Keep project dependencies isolated with a virtual environment.
- Use readable names and small, testable steps.
- For a data engineering task, always think about input format, data types, invalid records and output destination.
