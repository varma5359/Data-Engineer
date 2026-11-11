# Day 34 Interview Guide — Data Transformation and Business Rules

# 1. What is data transformation?
**Model answer:** Transformation converts extracted source data into a consistent, validated representation for downstream use. It includes type parsing, standardization, mapping, derived fields and business validation. I preserve raw data and document rules so results can be replayed and audited.

# 2. Raw vs curated data?
**Model answer:** Raw retains source representation and provenance. Curated applies approved business rules and quality checks. Keeping raw allows reprocessing when mapping or logic changes without losing the original evidence.

# 3. Why profile before cleaning?
**Model answer:** Profiling shows actual values, null rates, distinct values, ranges and duplicates. I avoid assuming what data looks like. For example, status counts may reveal "paid", "PAID " and an unknown code; I then define normalization/mapping rules with the owner.

# 4. Parsing vs validation?
**Model answer:** Parsing interprets representation as a type, such as string "501" to integer 501. Validation checks whether the typed value is allowed, such as customer ID > 0. A value can parse successfully and still violate business rules.

# 5. Why Decimal for money?
**Model answer:** Decimal supports base-10 decimal arithmetic suitable for financial values. Binary float can represent decimal fractions imprecisely. I parse from source text, define currency/scale/rounding and avoid silently converting invalid amounts to zero.

# 6. How do you handle unknown status?
**Model answer:** I normalize only approved formatting differences, then use a controlled mapping table. Unknown values go to an exception/reject path with source code and reason. I ask the business owner to approve mapping; I do not guess that unknown means pending or paid.

# 7. Explain a derived amount.
**Model answer:** If quantity=3 and unit price=100.00, gross is 300.00. If discount=10.00 per line, net=290.00. I clarify whether discount is per item, per line or percentage, currency and rounding. I validate discount range and use Decimal.

# 8. What is cross-field validation?
**Model answer:** It checks relationships among fields. Example: status SHIPPED requires shipped_at. Individual fields can be valid while the record is contradictory. Rules must be business-approved and tested.

# 9. How do duplicates affect aggregation?
**Model answer:** Duplicate order lines can inflate sum/count. I identify the correct grain and stable key, deduplicate or resolve conflicting versions before aggregation. I don't simply drop duplicates without a rule.

# 10. How do you reconcile after transformation?
**Model answer:** Input records equal accepted plus rejected. I compare keys, counts, totals and status distributions. If business rules intentionally change amount, I reconcile using the documented formula and exclusions. Count alone doesn't prove values are correct.

# 11. How do you version transformations?
**Model answer:** Record rule/mapping version, code version, source file/run ID and output version. When a rule changes, replay raw data under the new version, compare results and document which outputs were rebuilt.

# 12. Explain your project.
**Model answer:** “I transformed TechStore order records by profiling status, converting customer IDs and amounts, mapping partner status to canonical values, calculating gross/net and validating ranges. I separated accepted and rejected rows with reasons, reconciled counts and checked keys/totals. Unknown status was not silently mapped. I preserved raw input so the pipeline can be replayed.”

# 13. Follow-up prompts
- What if discount is greater than gross?
- What if amount is null?
- What if a status code changes tomorrow?
- What if two records have same key but different amounts?
- Why is uppercasing customer names dangerous?
- What if accepted count becomes zero?
- How do you prove transformation did not lose records?
- How would you test rounding?
- How do you replay after mapping correction?
- What would you log without exposing PII?

# 14. Project defence
Explain business rule, input evidence, code, output, rejects, reconciliation and rule version. State assumptions and ask for clarification when business meaning is ambiguous.
