# Day 7 Interview Preparation: Functions and Modules

**Mini-project:** Reusable Transformation Library

## 1. Core questions

1. **Why use functions in a pipeline?**  
   To isolate transformations, avoid duplicated logic, improve testing and make code easier to maintain.

2. **Parameter vs argument?**  
   A parameter appears in the function definition; an argument is the value passed during a call.

3. **Return vs print?**  
   Return sends a value to the caller. Print displays it but does not provide a reusable result.

4. **What is a module?**  
   A Python file that groups related code and can be imported by another file.

5. **What is a docstring?**  
   A string documenting a function, class or module's purpose and usage.

## 2. Difference-based questions

| Compare | Answer |
|---|---|
| Positional vs keyword argument | Matched by position vs parameter name |
| Local vs global variable | Defined in a function scope vs module scope |
| Function vs module | Reusable callable block vs file grouping code |
| `return` vs `yield` | Return a function result vs produce generator values over time |
| Import vs execute | Load module definitions vs run entry-point behavior |

## 3. Practical questions

6. **How do you prevent invalid input?**  
   Validate at the function boundary and raise a meaningful exception or return an explicit validation result.

7. **How do you organize reusable transformations?**  
   Keep related functions in a module, give each function a focused responsibility, document assumptions and import them from the pipeline entry point.

8. **Why avoid mutable default arguments?**  
   The same default object can be reused across calls, causing state to leak between executions. Use `None` and create a new object inside.

## 4. Scenario questions

### Scenario A: Revenue logic copied in three scripts
Create one tested transformation function and reuse it. Confirm all scripts use the same approved revenue definition.

### Scenario B: ImportError in a project
Check file name, import statement, package layout, working directory and active interpreter. Avoid naming project files after standard modules such as `json.py`.

### Scenario C: Function prints a value but downstream step receives None
Replace print-only behavior with `return`, then assign the returned value to the next pipeline variable.

## 5. Troubleshooting

- `NameError`: function or variable is not defined in the current scope.
- `TypeError`: wrong number or type of arguments.
- `ValueError`: argument value is invalid.
- `ImportError`: module or requested name cannot be imported.
- Unexpected shared list: inspect mutable defaults and global state.

## 6. Follow-up questions

- What is a pure function?
- What is function scope?
- Why use a main guard?
- How do you document expected input types?
- How would you test a transformation?
- When should a function raise an exception?

## 7. Mini-project defence

Explain each function's contract, input, output, validation and side effects. Show that the same transformation produces the same result for the same input and that invalid records fail visibly rather than being silently changed.
