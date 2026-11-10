# Day 7 — Functions, Modules and Reusable Code | Interview

**What is a function?** A named reusable block that accepts inputs and can return a result.

**Parameter vs argument?** Parameter appears in function definition; argument is the actual value passed during a call.

**Return vs print?** Return makes a result available to the caller; print displays it. Reusable transformations should generally return.

**Why modules?** Separate related responsibilities and allow imports/reuse, clearer ownership, and easier testing.

**What is a docstring?** Documentation string describing a function/module/class; useful for purpose, inputs, output, and constraints.

**How would you structure a small ETL?** Separate extraction, transformation, validation, loading, and orchestration; keep functions focused and pass data explicitly.

Scenario: two pipelines need identical date normalization. Put one tested implementation in a shared module, document expected input/output, and import it rather than copying code.

Follow-up: How do you test a function? Use known inputs and expected outputs, including invalid and boundary cases.