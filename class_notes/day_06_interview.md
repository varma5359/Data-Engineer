# Day 6 Interview Preparation: Python Introduction and Environment

**Mini-project:** Data Processing Utility

## 1. Core questions

1. **Why is Python used in data engineering?**  
   It supports file processing, API/database integration, automation, validation and orchestration. SQL remains important for relational transformations.

2. **What is the Python interpreter?**  
   The program that executes Python code.

3. **What is a virtual environment?**  
   An isolated project environment for Python packages, helping prevent dependency conflicts.

4. **What is the difference between a script and a notebook?**  
   A script is a saved program executed as a unit. A notebook is an interactive document containing code, output and explanatory text cells.

5. **What is dynamic typing?**  
   A variable name can refer to values of different types during execution; the type belongs to the value, not a fixed variable declaration.

## 2. Difference-based questions

| Compare | Answer |
|---|---|
| List vs tuple | Mutable vs immutable ordered collection |
| List vs set | Ordered/indexed collection vs unique elements without positional indexing |
| Dictionary vs list | Key-value lookup vs positional indexing |
| int vs float | Whole-number type vs floating-point decimal representation |
| `==` vs `is` | Value equality vs object identity |
| `None` vs empty string | Absence marker vs text containing zero characters |

## 3. Practical questions

6. **Why can an ID be stored as a string?**  
   It identifies an entity rather than representing a quantity. Converting it to an integer can remove leading zeros.

7. **How do you inspect a value's type?**  
   Use `type(value)`.

8. **When would you use a dictionary?**  
   When fields need to be accessed by meaningful keys, such as `order["status"]`.

9. **How do you handle a possibly missing dictionary key?**  
   Use `.get()` with an appropriate default, while still validating required fields.

10. **Why use a virtual environment?**  
    To isolate project dependencies and make setup reproducible.

## 4. Scenario questions

### Scenario A: Module works in terminal but not in VS Code
**Action:** Check the selected interpreter in VS Code, activate the project's virtual environment and verify the package is installed in that interpreter.

### Scenario B: Customer ID 00078 becomes 78
**Action:** Keep the identifier as text from ingestion through storage. Do not treat it as a numeric measure.

### Scenario C: A pipeline receives quantity as "four"
**Action:** Validate the value before integer conversion, quarantine invalid input and record the reason rather than silently substituting a number.

## 5. Troubleshooting

- `python is not recognized`: verify installation and PATH; on Windows try `py`.
- `ModuleNotFoundError`: confirm the active interpreter and install package into that environment.
- `KeyError`: inspect actual input keys and schema.
- `TypeError`: check the types involved in the operation.
- Wrong notebook output: rerun cells in execution order or restart the kernel and run all cells.

## 6. Follow-up questions

- What is a mutable object?
- Why are sets useful for uniqueness checks?
- What is a Python package?
- How do you pin dependency versions?
- What happens when a list index is out of range?
- Why should source data not be overwritten during exploration?

## 7. Mini-project defence

Explain how the Data Processing Utility represents orders, calculates line amounts and filters paid records. Discuss what must be added before production use: schema validation, duplicate handling, currency rules, logging, tests and persistent output.
