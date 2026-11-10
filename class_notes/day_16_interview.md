# Day 16 Interview Preparation: Data Validation and Schema Checking

**Mini-project:** Data Validation Engine

## 1. Core questions

1. **What is data validation?** Checking whether data meets defined structural, technical and business expectations.
2. **Schema validation vs business validation?** Schema checks fields/types/shape; business validation checks rules such as positive quantity or valid status.
3. **What is a data contract?** An agreement describing a dataset's schema, meaning, quality, ownership and change expectations.
4. **Why quarantine rejected records?** To preserve evidence, support correction and prevent invalid data from silently entering curated tables.
5. **What is schema evolution?** Controlled change to a dataset's structure over time.

## 2. Difference-based questions

| Compare | Answer |
|---|---|
| Null vs zero | Missing value vs numeric zero |
| Duplicate row vs duplicate key | Identical row vs repeated business identifier |
| Warning vs critical validation | Observable issue vs condition that blocks safe processing |
| Backward-compatible vs breaking change | Existing consumers continue vs consumer changes required |
| Technical vs business rule | Format/type constraint vs domain meaning |

## 3. Practical questions

6. **How do you validate uniqueness?** Define the table grain and business key, then check duplicates at that key.
7. **How do you handle invalid rows?** Preserve them with reason codes, apply approved thresholds and alert the owner.
8. **How do you validate timestamps?** Parse against an explicit format and timezone convention.
9. **What should happen when a required field disappears?** Treat it as a contract-breaking change and stop or quarantine the batch according to policy.

## 4. Scenario questions

### Scenario A: Source adds a new optional field
Review compatibility, update documentation and tests, and ensure downstream consumers tolerate it.

### Scenario B: Amount values become strings
Check source contract, parse safely, quarantine invalid values and coordinate with producer owner.

### Scenario C: Duplicate keys found
Check source behavior, update timestamps and table grain. Define whether to deduplicate, retain history or reject the batch.

## 5. Troubleshooting
- False duplicate alerts: key may not represent the correct grain.
- Many rejected rows: inspect producer release and schema drift.
- Date parsing failures: confirm format, locale and timezone.
- Null explosion: compare source version and upstream completeness.

## 6. Follow-up questions
- What is referential integrity?
- What is a schema registry?
- What is drift detection?
- What is a quarantine table?
- How do you version a data contract?
- How do you measure completeness?

## 7. Mini-project defence
Present the contract, validation categories, severity rules and rejection report. Explain how a downstream consumer is protected when a producer changes its schema.
