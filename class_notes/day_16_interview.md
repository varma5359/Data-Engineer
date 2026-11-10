# Day 16 Interview Preparation: Data Validation and Schema Checking

**Mini-project:** Data Validation Engine

## 1. Core questions

1. **What is data validation?**  
   Checking whether data meets defined structural, technical and business expectations before it is used downstream.

2. **Parsing vs schema validation vs business validation?**  
   Parsing reads the representation; schema validation checks fields and types; business validation checks rules such as positive quantity or approved status.

3. **What is a data contract?**  
   An agreement describing a dataset's schema, field meaning, quality expectations, ownership and change policy.

4. **Why validate before loading?**  
   Incorrect data can contaminate reports, models and downstream systems even when the pipeline code runs successfully.

5. **What is a required field?**  
   A field that must be present and contain an acceptable value according to the contract.

6. **What is referential integrity?**  
   A rule ensuring a reference such as customer_id points to an existing customer record.

7. **What is schema evolution?**  
   A controlled change to a dataset's structure over time, such as adding, removing or changing a field.

8. **Why keep rejected records?**  
   They provide evidence for investigation, correction and replay instead of hiding data loss.

## 2. Difference-based questions

| Compare | Answer |
|---|---|
| Schema vs business validation | Structure/type checks vs domain rules |
| Missing vs invalid value | Absent/blank vs present but unacceptable |
| Null vs zero | Unknown/absent value vs numeric zero |
| Reject vs quarantine | Refuse a record vs isolate it for investigation |
| Hard vs soft failure | Stop the batch vs continue under an approved policy |
| Accuracy vs validity | Reflects reality vs conforms to defined rules |

## 3. Practical questions

9. **How do you check required fields in a dictionary?**  
   Iterate over the required field list and check whether each key is absent, None or blank.

10. **How do you convert a raw amount safely?**  
    Check missing input, convert from the original string using Decimal for financial values and capture conversion errors.

11. **How do you validate an allowed status?**  
    Compare the value against an approved set of statuses.

12. **How do you detect duplicate IDs?**  
    Maintain a set of previously accepted business keys and reject or resolve repeats according to policy.

13. **Why store rejection reasons?**  
    They explain exactly why a row failed and allow data owners to correct the source.

14. **How do you reconcile input counts?**  
    Confirm total input rows equal accepted plus rejected rows, accounting for any explicitly documented exclusions.

## 4. Scenario questions

### Scenario 1: Amount is "unknown"

**Problem:** The amount field cannot be converted to a decimal.

**Strong answer:** Reject or quarantine the row with a clear conversion reason. Do not replace the value with zero unless the business contract explicitly defines that behavior.

### Scenario 2: Quantity is negative

**Problem:** A record has quantity -3.

**Strong answer:** Apply the documented range rule. If returns are represented as negative quantities in this system, the contract must define that exception; otherwise reject it.

### Scenario 3: A required field disappears

**Problem:** The upstream producer removes customer_id.

**Strong answer:** Treat it as a schema-breaking change, stop or quarantine according to severity, notify the producer and assess downstream impact.

### Scenario 4: Duplicate business key

**Problem:** Two records have the same order_id.

**Strong answer:** Check whether the source sent a duplicate or whether order_id is not the correct grain. Use an approved version/timestamp rule; do not silently discard one.

### Scenario 5: Rejection rate suddenly increases

**Problem:** Yesterday 1% of records failed; today 40% fail.

**Strong answer:** Compare rejection reasons, source version and batch metadata. Alert the owner and investigate a source schema or upstream quality change before publishing misleading data.

## 5. Troubleshooting questions

15. **Why can int conversion fail?**  
    The value may be blank, contain non-numeric text or contain a decimal representation that is not an integer.

16. **Why should validation order matter?**  
    Required-field checks should happen before conversion; range checks should happen after successful conversion.

17. **Can a record have multiple validation errors?**  
    Yes. Collect independent errors so the source owner can correct the row efficiently.

18. **What should happen to accepted and rejected rows?**  
    Accepted rows continue to the approved next stage; rejected rows are retained with reasons and traceable source metadata.

## 6. Mini-project explanation

"I built a validation engine for incoming order records. It checked required fields, converted numeric values, enforced positive IDs and quantity, checked non-negative amount and allowed statuses, and detected duplicate order IDs. It separated accepted records from rejected records with reasons and reconciled the counts."

## 7. Follow-up questions

- Would you fail the entire batch if one row is invalid?
- How do you version a data contract?
- How do you validate foreign keys across two datasets?
- How would you alert on a sudden quality drop?
- How do you prevent rejected records from being loaded later by mistake?
- How do you make validation deterministic and testable?

## 8. Quick revision

- Validate structure, types and business meaning separately.
- Convert raw values safely.
- Keep rejected records with reasons.
- Use the correct business key for duplicate checks.
- Reconcile input, accepted and rejected counts.
- Treat schema changes as controlled engineering changes.


---

## 7. Follow-up interview questions

16. What is data validation? Checking whether data meets structural and business expectations.
17. Parsing versus schema validation? Parsing checks whether representation can be read; schema validation checks fields and types.
18. What is a business-rule validation? A check that a value is acceptable under business meaning, such as positive quantity.
19. What is a required field? A field that must be present and valid for a record to be accepted.
20. Why distinguish null, empty string and zero? They represent different states and can have different business meanings.
21. What is a duplicate key? A repeated identifier that should uniquely identify a record under the agreed data model.
22. Why preserve rejected rows? For correction, audit, replay and source-owner feedback.
23. What is schema drift? An unexpected change in the structure or types of incoming data.
24. What is a data contract? An agreement about data structure, meaning, quality expectations and change behavior between producers and consumers.
25. Should unknown columns always be rejected? Not necessarily; policy depends on whether additional fields are permitted and whether they affect downstream processing.
26. How validate a date field? Parse using the agreed format, reject invalid values and verify any business date constraints.
27. How measure validation quality? Track input, accepted and rejected counts and reason categories; ensure the counts reconcile.
28. What is a quality threshold? A defined limit, such as maximum rejection percentage, that determines whether processing can proceed.
29. Why not replace all invalid amounts with zero? It hides source errors and can distort financial totals.
30. Where should validation happen? At ingestion boundaries and again where downstream transformations introduce new constraints.

## 8. Scenario questions

A source adds an optional column. Check the contract and ensure consumers tolerate additive changes where intended.

A required identifier disappears. Fail or quarantine according to the contract because records cannot be reliably keyed.

The rejected percentage rises sharply. Stop or alert based on the quality threshold and investigate the source change before publishing output.

## 9. Mini-project defense

- What fields were mandatory?
- Which type conversions could fail?
- Which business rules were applied?
- How were duplicates detected?
- How do accepted and rejected counts reconcile?

## 10. Readiness rubric

| Skill | Ready when the student can… |
|---|---|
| Parsing | Detect values that cannot be converted |
| Schema | Check required fields and types |
| Business rules | Validate allowed values and ranges |
| Duplicates | Detect repeated business keys |
| Rejections | Preserve reasons and source context |
| Contracts | Explain schema drift and quality thresholds |
