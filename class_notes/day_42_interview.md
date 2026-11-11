# Day 42 Interview Guide — Slowly Changing Dimensions

## 1. What is a Slowly Changing Dimension?

A Slowly Changing Dimension is a strategy for managing changes to dimension attributes over time. The business requirement determines whether the warehouse overwrites the old value, preserves full history or stores a limited previous value.

## 2. Explain SCD Type 1.

Type 1 overwrites the old attribute with the new value. It is useful for corrections or attributes where only the current value matters. It does not preserve the previous value in the dimension row, so it cannot independently answer historical questions about that attribute.

## 3. Explain SCD Type 2.

Type 2 preserves history by creating a new dimension row when a tracked attribute changes. The old row is closed and the new row becomes current. A common design includes a surrogate key, business key, effective start, effective end and current flag.

## 4. Explain SCD Type 3.

Type 3 stores a limited previous value in an additional column. For example, current_city and previous_city can support a current-versus-previous comparison. It does not preserve an unlimited timeline.

## 5. Compare the three types.

Type 1 overwrites, Type 2 adds a historical version, and Type 3 keeps limited previous state in columns. Type 2 is appropriate when reports must reconstruct historical attributes. Type 1 is appropriate for corrections or current-only requirements. Type 3 is appropriate for a limited comparison.

## 6. Why does Type 2 need a surrogate key?

The same business key can have multiple historical versions. A surrogate key uniquely identifies each dimension version so a fact can reference the version valid at the event time.

## 7. What are effective dates?

Effective dates define when a dimension version is valid in the business. The warehouse should use a consistent boundary convention. For timestamp models, half-open intervals are often useful: start included, end excluded.

## 8. What is a current flag?

A current flag identifies the version currently valid according to the model's rule. A validation should ensure no business key has more than one current row.

## 9. What is a late-arriving dimension change?

It is a business change received by the warehouse after its effective date. The engineer must distinguish when the change became true from when it arrived, insert the correct historical version and assess whether facts already loaded need correction.

## 10. How do facts use a Type 2 dimension?

The fact loader finds the dimension version valid for the fact event timestamp and stores that version's surrogate key. Joining facts to the current row by business key can rewrite historical interpretation and is usually wrong when historical attributes are required.

## 11. How do you avoid creating a new version on every load?

Compare the incoming tracked attributes with the current dimension row. Only create a new version when a tracked business attribute actually changes. Do not use ingestion timestamp alone as evidence of a business change.

## 12. How do you handle two changes on the same day?

Use the source's reliable change timestamp or sequence to order the changes. If the source provides only a date, the business must define whether multiple same-day states can be distinguished. Do not invent an ordering. Use a timestamp or sequence where available and define interval boundaries consistently.

## 13. What happens if the old row closes but the new row fails?

If the operations are in one transaction, roll back so the dimension remains in its previous valid state. If they were committed separately, the dimension may have no current row and needs controlled repair. This is why transaction boundaries matter.

## 14. How do you detect overlapping versions?

Compare rows for the same business key and identify intersecting effective ranges. For half-open intervals, overlap occurs when each row starts before the other row ends. A valid Type 2 design should return no unintended overlaps.

## 15. What is the difference between effective time and load time?

Effective time is when the business change became true. Load time is when the warehouse received or stored it. A late-arriving change has an effective time earlier than its load time.

## 16. Scenario: the customer city changes from Hyderabad to Bengaluru. The manager wants old orders to remain associated with Hyderabad. Which SCD type?

Type 2. Create a new customer version and ensure historical facts reference the dimension key valid at order time.

## 17. Scenario: a spelling correction should apply to all reports, including historical reports. Which type?

Type 1 is usually appropriate because the requirement is to correct the value rather than preserve the incorrect spelling as business history. Audit requirements may still require a separate change log.

## 18. Scenario: only current and previous sales territory are needed. Which type?

Type 3 may be appropriate if the business confirms that older history is not required.

## 19. Scenario: the dimension has two current rows.

I would inspect the source changes and load concurrency, identify which row is valid, and repair the history under a controlled transaction. I would add a uniqueness rule or validation to prevent the issue from recurring.

## 20. Explain your customer history project.

I defined customer_id as the business key and city as the tracked attribute. I created a Type 2 dimension with a surrogate key, effective dates and current flag. When the city changed, I closed the old row and inserted a new version in one transaction. I queried the dimension by event date and validated current rows and date-range overlaps.

## Interview traps

- Calling Type 1 a history-preserving method.
- Saying Type 2 always means one new row for every incoming record.
- Using the current dimension row for every historical fact.
- Confusing source arrival date with business effective date.
- Forgetting same-day change ordering.
- Closing the old version and inserting the new version in separate commits.
- Assuming a current flag alone proves history ranges are valid.

## Readiness checklist

- [ ] Explain all three SCD types.
- [ ] Defend the business requirement behind the selected type.
- [ ] Explain surrogate keys and effective dates.
- [ ] Describe late-arriving changes.
- [ ] Explain transactional updates.
- [ ] Describe overlap and current-row validation.
