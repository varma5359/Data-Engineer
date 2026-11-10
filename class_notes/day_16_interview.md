# Day 16 — Data Validation | Interview

**Schema validation?** Confirm expected fields, types, and constraints before processing/publishing.

**Completeness vs validity?** Completeness checks presence; validity checks whether a value conforms to a rule.

**What is a data contract?** Agreement describing schema, semantics, quality, ownership, and compatibility.

**Reject vs quarantine?** Reject stops unsafe batch; quarantine isolates invalid records if policy allows.

**What is reconciliation?** Compare source/destination counts, totals, and keys to detect missing or altered data.

**Scenario: 2% rows invalid.** Decide using severity and agreed thresholds; report reasons, preserve evidence, protect valid output, notify owner, and don't silently drop rows.