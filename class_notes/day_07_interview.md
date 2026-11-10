# Day 7 Interview Preparation: Functions and Modules

**Mini-project:** Reusable Transformation Library

## 1. Core questions

1. **What is a function?**  
   A named reusable block of code that performs a focused task.

2. **Why use functions in a pipeline?**  
   To avoid duplicated logic, isolate transformations, improve testing and make code easier to maintain.

3. **What is a parameter?**  
   A variable named in a function definition.

4. **What is an argument?**  
   A value supplied when calling a function.

5. **What does return do?**  
   It sends a result back to the caller and ends that function execution.

6. **Print vs return?**  
   Print displays a value; return makes a value available to the calling code.

7. **What is a default parameter?**  
   A parameter with a fallback value used when the caller omits it.

8. **What is a keyword argument?**  
   An argument supplied using its parameter name.

9. **What is local scope?**  
   A variable created inside a function is normally available only within that function.

10. **What is a module?**  
    A Python file containing reusable code that can be imported.

11. **What is a package?**  
    A way to organize related Python modules for reuse and distribution.

12. **Why split a project into modules?**  
    It separates responsibilities and makes code easier to understand, test and maintain.

13. **What happens if a function has no return statement?**  
    It returns `None` by default.

14. **Why validate function inputs?**  
    To reject invalid values early and prevent incorrect downstream results.

15. **What is a virtual environment?**  
    An isolated environment for the dependencies of one project.

## 2. Difference questions

| Concept | Difference |
|---|---|
| Parameter / argument | Definition name / supplied value |
| Print / return | Display / send result to caller |
| Positional / keyword argument | Matched by order / matched by name |
| Function / module | Reusable code block / Python file containing definitions |
| Module / package | One importable module / organized collection of modules |
| Local / global variable | Function-local name / module-level name |
| Library / framework | Reusable tools / framework often controls application structure |

## 3. Practical coding questions

**Q1. Write a function to calculate order amount.**

```python
def calculate_amount(quantity, price):
    return quantity * price

print(calculate_amount(2, 250))
```

Output: `500`.

**Q2. Use a default discount.**

```python
def apply_discount(amount, rate=0.10):
    return amount * (1 - rate)

print(apply_discount(1000))
```

Output: `900.0`.

**Q3. Normalize a status.**

```python
def normalize_status(status):
    return status.strip().upper()

print(normalize_status(" shipped "))
```

Output: `SHIPPED`.

**Q4. Reject invalid quantity.**

```python
def validate_quantity(quantity):
    if quantity <= 0:
        raise ValueError("quantity must be positive")
    return quantity

print(validate_quantity(2))
```

Output: `2`.

**Q5. Return two summary values.**

```python
def summarize(values):
    return sum(values), len(values)

total, count = summarize([100, 200, 300])
print(total, count)
```

Output: `600 3`.

## 4. Scenario-based questions

1. **The same transformation is copied into four pipeline steps. What would you do?**  
   Move the shared business rule into one function, test it and call it from each step.

2. **A function prints a total but the next pipeline step needs to use it. What is wrong?**  
   It should return the total. Printed output is not a reusable program result.

3. **A function accepts a negative price. How do you protect it?**  
   Validate the input and raise a clear exception before calculation.

4. **An import fails with ModuleNotFoundError. What do you inspect?**  
   Confirm the module filename, project folder, working directory, import spelling and active Python environment.

5. **A project works on one computer but not another. Why?**  
   Dependencies or Python versions may differ. Use a virtual environment and record dependencies.

## 5. Follow-up questions

- Can a function return more than one value?
- What does a function return if there is no return statement?
- Why should functions generally have one clear responsibility?
- What is a docstring?
- What is the difference between importing a full module and importing selected names?
- Why are mutable default arguments dangerous?
- How would you test a transformation function?

## 6. Strong revision points

- Functions reduce duplication and make business rules reusable.
- Parameters define inputs; arguments supply values.
- Return values connect one pipeline step to another.
- Validate assumptions at function boundaries.
- Modules organize reusable code.
- Virtual environments and dependency files support reproducible execution.
