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


---

## 7. Deeper follow-up questions and model answers

16. **What is the difference between a variable and a value?**  
    A value is the data itself, such as the integer `10`. A variable is a name that refers to that value. In `quantity = 10`, `10` is the value and `quantity` is the name.

17. **Does Python require a variable type declaration?**  
    No. Python determines the type at runtime. However, data engineers should still understand and validate types because source data may be inconsistent.

18. **What is the difference between mutable and immutable objects?**  
    Mutable objects can be changed after creation, such as lists and dictionaries. Immutable objects cannot be changed in place, such as strings, integers and tuples.

19. **Why might a list of dictionaries be useful?**  
    It represents a small collection of records. Each dictionary contains named fields, and the list contains multiple records. It is convenient for learning and small in-memory tasks, but not a substitute for scalable storage.

20. **What is zero-based indexing?**  
    The first element has index 0, the second has index 1, and so on.

21. **What does slicing do?**  
    It selects a portion of a sequence. In `values[start:stop]`, the start is included and the stop is excluded.

22. **What is the difference between `append()` and `extend()`?**  
    `append()` adds its argument as one item. `extend()` adds each element from an iterable.

23. **Why use `dict.get()`?**  
    It safely reads an optional key and can return a default value when the key is absent. It does not replace schema validation when a field is mandatory.

24. **What does `None` mean?**  
    It represents the absence of a value. It is different from zero, an empty string and False.

25. **Why is `bool("False")` True?**  
    A non-empty string is truthy in Python. The text "False" is not the Boolean value False. Parse source flags according to an explicit accepted-value rule.

26. **Why can floating-point calculations be surprising?**  
    Many decimal fractions cannot be represented exactly in binary floating point. For financial values, use an appropriate decimal strategy and agreed rounding rules.

27. **What does `+=` mean?**  
    It updates a variable using addition, such as `total += amount`, which is equivalent to `total = total + amount` for this example.

28. **What is the difference between a syntax error and a runtime exception?**  
    A syntax error prevents Python from parsing the code. A runtime exception occurs while validly parsed code is executing.

29. **How would you investigate a wrong total?**  
    Inspect each input's type and value, verify the calculation rule, print or inspect intermediate results, check whether invalid or duplicate records were included, and compare the result with a small manually calculated sample.

30. **Why should a pipeline not convert every invalid amount to zero?**  
    Zero may be a valid business amount. Replacing invalid input with zero hides data-quality problems and can make financial totals incorrect. Preserve the raw value and record a rejection reason or apply an approved business rule.

## 8. Output-tracing interview drill

**Question:** Predict the output before running the code.

```python
prices = [100, 200, 50]
prices.append(150)

total = 0
for price in prices:
    total += price

print(len(prices))
print(total)
print(prices[1:3])
```

**Answer:**

```text
4
500
[200, 50]
```

Reasoning:
- `append(150)` adds one element, so the list has four values.
- The total is 100 + 200 + 50 + 150 = 500.
- Slice `[1:3]` includes indexes 1 and 2, not index 3.

## 9. Mini-project defense questions

1. Why did you represent the orders as a list of dictionaries?
2. Which fields are required for the amount calculation?
3. What happens if quantity is stored as text?
4. How would you handle a missing order ID?
5. How did you verify the total sales figure?
6. Why is the in-memory list suitable for this small exercise but not necessarily for millions of records?
7. What would you change before loading the records into a database?

**A strong project explanation:**  
"I created a small in-memory order dataset using a list of dictionaries. Each dictionary represents one order. I calculated line amounts from quantity and unit price, checked for invalid quantities, and aggregated valid amounts. I used this small example to understand Python values and collections. For larger data, I would read from a file or database, validate records systematically, process in manageable batches and write results to a durable destination."

## 10. Day 6 readiness rubric

| Skill | Ready when the student can… |
|---|---|
| Environment | Check Python version and run a saved script |
| Variables and types | Explain a value's type and convert compatible values |
| Collections | Choose a list or dictionary for a simple record task |
| Control flow | Read a loop and predict its output |
| Data reasoning | Calculate line amounts and totals correctly |
| Debugging | Recognize common beginner exceptions |
| Communication | Explain the mini-project without memorizing a script |
