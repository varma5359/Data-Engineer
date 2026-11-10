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


---

## 7. Follow-up interview questions

16. **What happens when a function is defined?** Python creates the function object, but its body runs only when the function is called.
17. **What is a parameter?** A named input in a function definition.
18. **What is an argument?** The actual value supplied when calling a function.
19. **Why use return?** It gives a result back to the caller so later steps can use it.
20. **Can a function return more than one value?** It can return a tuple containing multiple values, which can be unpacked by the caller.
21. **What is a default parameter?** A parameter with a value used when the caller omits that argument.
22. **What is a keyword argument?** An argument supplied by naming its parameter, such as `rate=0.10`.
23. **What is local scope?** Names created inside a function are normally available only inside that function.
24. **What is a module?** A Python file that groups reusable code.
25. **What is a package?** A way to organize related modules into a namespace/package structure.
26. **Why avoid copying the same transformation?** One implementation is easier to test, review and update consistently.
27. **What is a pure function?** A function whose result depends on its inputs and that has no observable side effects. It is often easier to test.
28. **What is a side effect?** A change outside the function's returned result, such as writing a file or changing a global variable.
29. **What causes ModuleNotFoundError?** Python cannot locate the requested module in the active import search path.
30. **How would you test a transformation function?** Supply known inputs, compare the returned result with an expected result, and include boundary and invalid cases.

## 8. Output-tracing drill

```python
def add_tax(amount, rate=0.05):
    return amount + (amount * rate)

first = add_tax(100)
second = add_tax(100, 0.10)
print(first)
print(second)
```

Expected output:

```text
105.0
110.0
```

The first call uses the default rate of 5%. The second call overrides it with 10%.

## 9. Mini-project defense

- Why did you put transformations in a separate module?
- Which function returns a value and why?
- How would you handle a missing quantity?
- What test inputs would you use for a quantity validation function?
- How does a reusable function help when a business rule changes?

**Strong explanation:** "I separated small order transformations into a module so the main script can import and reuse them. Each function has one clear responsibility and returns a result that can be tested independently."

## 10. Readiness rubric

| Skill | Ready when the student can… |
|---|---|
| Define and call | Write and execute a simple function |
| Inputs | Explain parameters and arguments |
| Results | Explain print versus return |
| Defaults | Use default and keyword arguments |
| Modules | Import a function from a second file |
| Reuse | Explain how reuse reduces inconsistent business logic |
