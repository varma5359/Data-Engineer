# Day 35 Interview Guide — Data Cleaning and Schema Evolution

**Project:** TechStore Data Cleaning Framework  
**Audience:** Fresher / Junior Data Engineer  
**How to use this guide:** Read the question, answer aloud in your own words, compare with the model answer, and then identify the exact evidence in your project. Do not memorize isolated definitions.

---

## Part 1 — Core concepts with complete answers

### 1. What is data cleaning?

**Model answer:** Data cleaning is the controlled process of identifying data-quality problems and applying approved actions so the resulting data is fit for its intended use. Actions can include standardizing formatting, converting valid values, correcting values when an authoritative rule exists, or rejecting and quarantining records that cannot safely be used.

For example, TechStore receives an order file where a status is written as `" paid "`. If the source contract confirms that surrounding spaces are accidental, trimming the value is safe. If the amount is `"unknown"`, guessing an amount is not safe; the row should be rejected or quarantined with a reason.

**Why this answer is strong:** It explains that cleaning is governed by business rules, not arbitrary editing.

**Follow-up:** Why not delete every bad row?  
**Answer:** Deletion hides data loss. Retaining rejected rows and reasons makes the process auditable and allows correction and replay.

### 2. Explain profiling, validation, cleaning and transformation.

**Model answer:**
- **Profiling** describes what arrived: row count, column names, null counts, distinct values, ranges and duplicate patterns.
- **Validation** checks whether data satisfies defined rules.
- **Cleaning** applies approved corrections or rejection decisions.
- **Transformation** changes the representation or derives business values, such as converting a date string into a date or calculating net amount.

The order matters. Profile before changing the data so the original condition is recorded. Validate against the contract. Apply only approved cleaning rules. Then transform into the target representation.

**Example:** First count blank order IDs; next validate that order_id is required; then reject rows with a blank ID; finally convert valid order_date strings into dates.

### 3. What are data-quality dimensions?

**Model answer:** Common dimensions are completeness, validity, uniqueness, consistency, accuracy, timeliness and integrity. Completeness asks whether required values exist. Validity asks whether values follow allowed formats or ranges. Uniqueness checks business identity. Consistency checks whether related fields agree. Accuracy asks whether values represent reality. Timeliness checks whether data arrives when needed. Integrity checks relationships such as a valid customer reference.

A pipeline can automatically test completeness and format validity. It cannot always prove real-world accuracy without an authoritative source.

### 4. What is the difference between a null, blank string and zero?

**Model answer:** A null represents an absent or unknown value. A blank string is a present text value containing no characters (or possibly whitespace). Zero is a numeric value. They must not be treated as interchangeable.

For example, a missing discount may mean “not supplied”; a discount of zero means “no discount”. Replacing every null with zero changes business meaning and can distort reports.

### 5. How do you handle missing values?

**Model answer:** First identify the field's business meaning and whether it is required. A missing primary business key usually makes a record unusable and should be rejected. A missing optional comment may remain null. A default should be applied only when the business owner defines a meaningful default. I record the number of affected rows and the action taken.

**Follow-up:** Would you fill a missing customer_id with 0?  
**Answer:** No. That invents an identifier and may create a false relationship. Reject or quarantine the record until the source can provide the correct ID.

### 6. How do you identify duplicates?

**Model answer:** I first define the business key or event identity. Then I inspect duplicate counts using that key and compare the full records, timestamps and source semantics. Two identical rows may be a retransmission, but two rows with the same order_id may represent an update or multiple order events. I do not blindly keep the first row. I apply a documented winner rule, such as latest source update timestamp, only when that rule is approved.

### 7. What is the difference between duplicate rows and duplicate business keys?

**Model answer:** Duplicate rows have the same values across all selected columns. Duplicate business keys share an identifier that should identify one entity or event, even if other fields differ. For example, two records can have the same order_id but different amounts. They are not identical rows, but they conflict on the order identity and need investigation.

### 8. What is a data contract?

**Model answer:** A data contract is an agreement between the data producer and consumer describing the dataset's schema, field meanings, data types, requiredness, constraints, delivery expectations, ownership and change process. It turns assumptions into explicit rules.

For TechStore, the contract may say order_id is a required string, amount is a non-negative decimal, status must be one of PAID/PENDING/CANCELLED, and order_date must be a valid business date.

### 9. What is schema drift?

**Model answer:** Schema drift is a difference between the incoming structure and the structure the consumer expects. Examples include a missing column, unexpected column, renamed field or changed data type. Drift can be intentional or accidental; the pipeline should detect it rather than silently assuming the change is safe.

### 10. What is schema evolution?

**Model answer:** Schema evolution is the controlled management of planned schema changes over time. It includes versioning, compatibility assessment, producer and consumer coordination, migration rules and tests. Schema drift is an observed difference; schema evolution is the governed process for accepting and managing a change.

### 11. Is adding a new column always backward-compatible?

**Model answer:** No. A consumer that ignores unknown optional columns may continue working, but a strict schema validator may reject the file. A positional reader may interpret later fields incorrectly. Adding a required field can also break older producers that do not send it. Compatibility depends on how producers and consumers read the schema.

### 12. How do you detect a renamed column?

**Model answer:** Compare the incoming field set with the expected contract. A rename often appears as one expected column missing and one unexpected column present. I do not automatically map the new name based only on spelling. I confirm meaning and type with the source owner, document an approved mapping, version the contract and test downstream consumers.

### 13. What is quarantine?

**Model answer:** Quarantine is a controlled location for records or files that cannot safely enter the trusted dataset. It should retain original values, rejection reasons, source filename or event ID, ingestion run ID and relevant timestamps. It should support controlled investigation and replay, with suitable access and retention controls.

### 14. Why preserve raw data?

**Model answer:** Raw data is evidence of what the source actually sent. It supports audit, debugging, replay, comparison after cleaning-rule changes and investigation of source defects. Curated data should be stored separately so that cleaning never destroys the original evidence.

### 15. How would you clean an amount column?

**Model answer:** I inspect the raw values first. I define accepted formats with the business owner, then parse values into a suitable decimal representation. I check invalid strings, negative values if prohibited, nulls and precision. I avoid floating-point arithmetic for financial amounts when exact decimal behavior is required. Invalid values are rejected with a reason rather than silently converted to zero.

### 16. Why can blindly calling drop_duplicates be dangerous?

**Model answer:** It removes exact duplicate rows but does not understand business identity or event semantics. It may remove legitimate repeated events, or fail to resolve conflicting records with the same business key. The correct process defines the key, determines whether duplicates are retransmissions or updates, applies a documented rule and records what was removed or quarantined.

### 17. What is backward compatibility?

**Model answer:** A new schema is backward-compatible when existing consumers can continue reading the changed data without breaking. For example, adding an optional field may be compatible with consumers that ignore unknown fields. It is not automatically compatible with strict or positional consumers.

### 18. What is forward compatibility?

**Model answer:** A consumer is forward-compatible when it can handle data produced by a newer schema, often by safely ignoring fields or values it does not understand. The exact guarantee depends on the format and consumer implementation. Compatibility must be tested, not assumed.

### 19. How do you prove that cleaning did not lose records?

**Model answer:** I reconcile the input count against accepted and rejected records, while accounting for any documented aggregation or deduplication rule. I also record rule-level rejection counts and compare key totals where appropriate. For a one-input-row-to-one-decision process, input rows should equal accepted rows plus rejected rows.

**Example:** 1,000 input rows, 940 accepted and 60 rejected. The reconciliation is 1,000 = 940 + 60. If it does not balance, I investigate before publishing.

### 20. How would you handle an unexpected column in a daily file?

**Model answer:** I compare the received schema with the contract and classify the extra column. If the contract allows unknown optional fields, I may preserve and ignore it for this consumer while recording the drift. If the schema is strict or the new field changes meaning, I quarantine or fail the file and contact the producer. I never silently drop a potentially important field without a policy.

---

## Part 2 — Project-based questions

### 21. Explain your TechStore Data Cleaning Framework end to end.

**Model answer:** The project reads a reproducible partner order file, preserves the raw input, profiles row and column characteristics, and applies a documented contract. It standardizes approved formatting, parses values, validates required fields and business rules, and records every rejection reason. Accepted and rejected outputs are written separately. The final report reconciles input, accepted and rejected counts and reports schema differences. The important design decision is that invalid data is visible and explainable rather than silently discarded.

### 22. Why did you reject every record sharing a duplicate order key in the practice project?

**Model answer:** The exercise deliberately avoids inventing a winner. If two records share a key but disagree on values, keeping the first or last would be an arbitrary business decision. Rejecting the conflicting key group makes the conflict visible. In production, I would ask the business owner whether latest update timestamp, source priority or another rule defines the authoritative record.

### 23. What evidence would you show in a code review?

**Model answer:** I would show the input sample, declared contract, profiling summary, cleaning rules, accepted output, rejected output with reasons, schema comparison, reconciliation result and rerun behavior. I would also show tests for missing IDs, invalid amounts, duplicate keys and unexpected columns.

### 24. What would you improve before production?

**Model answer:** I would externalize and version the contract, add automated tests, capture source/run metadata, use immutable raw storage, add access controls and retention policies, publish quality metrics, define schema-change approval, and make reprocessing idempotent. For large datasets, I would process in chunks and avoid collecting all records in memory.

---

## Part 3 — Scenario questions with reasoning

### 25. The source adds a column called loyalty_tier. What do you do?

**Strong answer:** I compare the received schema with the current contract. I check whether the field is optional, whether its meaning and type are documented, and whether downstream readers tolerate additional fields. I preserve the raw file and record the drift. If the contract permits optional additions, I can accept it under that policy; otherwise I quarantine or fail the delivery until the producer and consumer agree on a versioned change.

### 26. A customer_id is missing on 2% of records. Do you fill it?

**Strong answer:** Not automatically. I first establish whether customer_id is required and whether a reliable reference source can resolve it. If no authoritative mapping exists, I quarantine or reject those records with a reason. I report the impact and ask the source owner to correct the feed. A fabricated ID would damage referential integrity.

### 27. The same order_id arrives twice with different amounts. What do you do?

**Strong answer:** I inspect source update timestamps, event IDs and delivery semantics. I determine whether the second record is a legitimate update or a conflict. I apply a latest-version rule only if it is part of the contract. Otherwise, I quarantine the conflict and preserve both source records for investigation.

### 28. A new schema removes a column that a downstream report uses.

**Strong answer:** This is a breaking change. I stop or quarantine the incompatible feed before publishing incomplete data, identify impacted consumers, notify the producer, and agree on a migration or compatibility window. I test the new version end to end and update the contract and lineage documentation.

### 29. The pipeline accepted 100% of rows, but revenue is 40% lower than yesterday.

**Strong answer:** Passing row-level validation does not prove business correctness. I compare source and target row counts, revenue totals, date coverage, currency, filters, duplicate handling and late-arriving data. I inspect source changes and transformation logic. I do not publish or declare success until the unexpected variance is explained or explicitly accepted.

### 30. A manager asks you to replace all missing amounts with zero so the report can run.

**Strong answer:** I explain that zero means a known amount of zero, while null means unknown or missing. Replacing null with zero changes the meaning and may understate revenue. I ask for a business-approved rule. Until then, I preserve the missing value and expose the affected count or quarantine records according to the contract.

---

## Part 4 — Practical revision checklist

Before the interview, make sure you can:
- explain the difference between profiling, validation, cleaning and transformation;
- distinguish null, blank and zero;
- define a business key before deduplication;
- explain why raw data and rejected records are retained;
- detect missing, extra, renamed and type-changed columns;
- explain schema drift versus planned schema evolution;
- reconcile input, accepted and rejected counts;
- walk through the TechStore project without reading the code;
- explain at least two trade-offs and two failure scenarios.

## Part 5 — Self-assessment

Score each item from 0 to 2: 0 = cannot explain, 1 = partial explanation, 2 = can explain with a project example.

| Skill | Score |
|---|---:|
| Data-quality dimensions | /2 |
| Profiling and validation | /2 |
| Null and duplicate handling | /2 |
| Contract and schema drift | /2 |
| Quarantine and raw retention | /2 |
| Reconciliation | /2 |
| Project explanation | /2 |
| Scenario reasoning | /2 |

**Interpretation:** 13–16: interview-ready foundation; 9–12: revise weak topics and rerun project drills; 0–8: revisit class notes and complete the workbook again.
