# Day 35 Interview Guide — Data Cleaning and Schema Evolution

**Project:** TechStore Data Cleaning Framework  
Practice each answer aloud, then point to project evidence.

## 1. What is data cleaning?
Data cleaning identifies quality problems and applies approved rules to correct, standardize, reject or quarantine records. It is not simply deleting rows or filling every null. In TechStore we normalize status, parse amounts, validate IDs and retain rejected records with reasons.

**Follow-up — prove no data disappeared:** Reconcile input count against accepted plus rejected, then inspect rule-level counts.

## 2. Profiling vs validation vs cleaning
Profiling describes data before changes (shape, types, nulls, duplicates, ranges). Validation checks a rule and reports pass/fail. Cleaning applies the approved action. Profile first to preserve baseline evidence.

## 3. What is schema?
The agreed structure of a dataset: field names, types, requiredness and constraints, often including meaning and relationships. It is the interface between producer and consumer.

## 4. Schema drift vs evolution
Drift is an unexpected difference from the contract. Evolution is a planned, versioned change with compatibility and consumer impact assessed. A surprise rename is drift; an approved rename with migration and tests is evolution.

## 5. How detect a renamed column?
Compare received names/types with the contract. A rename appears as missing expected plus extra received field. Confirm semantics with source owner, document mapping and test downstream. Never infer equivalence from spelling alone.

## 6. Is adding a column always backward-compatible?
No. Name-based consumers that ignore unknown optional fields may tolerate it; strict validators and positional readers may fail. Added required fields can break old producers. Test consumers.

## 7. What is a data contract?
An agreement on structure, field meaning, types, requiredness, quality, delivery, ownership and change process. It prevents surprise changes and clarifies response responsibilities.

## 8. How handle nulls?
Understand whether null means unknown, not applicable, not collected or source defect. Define field-specific policy: required IDs may be rejected; optional values may remain null; defaults need business meaning. Zero is not the same as unknown.

## 9. How identify duplicates?
Define business/event identity and source semantics first. Same key may be retransmission, update or legitimate multiple event. Determine authoritative version/time using approved rules; preserve conflicts for investigation. Identical payload does not always mean duplicate.

## 10. What is quarantine?
A controlled location for records that cannot safely be accepted. Retain original values, reasons, source/run metadata and replay context. Apply access and retention controls.

## 11. Why preserve raw data?
It is evidence of what source sent; supports audit, debugging, replay and comparison after rule changes. Curated outputs must be separate.

## 12. How clean amount?
Inspect source values and types; parse safely; count conversion failures; apply domain constraints. Negative amounts can be refunds/reversals, so consult business semantics. Preserve original value and rejection reason.

## 13. Idempotency?
Repeated processing of the same source should not create duplicate/inconsistent effects. Use stable source identity, run IDs, keys, deterministic outputs and safe overwrite/merge policy. It matters for retry/replay.

## 14. Backward vs forward compatibility?
Backward: newer producer/schema remains usable by existing consumers under rules. Forward: older consumer can handle newer producer data. Neither is automatic; unknown fields, required fields and type changes may break.

## 15. Handle breaking schema change in production
Stop/quarantine affected batch if unsafe; capture schema diff and samples; notify owner/consumers; agree mapping or contract version; test old/new; deploy in controlled order; replay safely; reconcile and document approval.

## 16. Prove project works
Show raw input, profile, rules, accepted/rejected outputs, reason codes, assertions, schema-drift simulation and reconciliation. Explain each record. Notebook success alone is not correctness proof.

## 17. Improve for production
Multiple reject reasons, versioned contracts, immutable raw storage, structured run metadata, secure quarantine, configurable rules, automated tests, alerts, bounded processing and safe replay. Separate business policy from implementation.

## 18. Scenario: supplier adds a field
Compare schemas; confirm optionality, meaning and type; assess consumers. If approved, update contract/version and tests. Otherwise stop/quarantine according to severity and contact owner.

## 19. Scenario: same ID, different amount
Do not keep first automatically. Inspect event/version/update time and source semantics. It may be update/correction or defect. Preserve evidence, apply approved authoritative rule or quarantine conflict, and report.

## 20. Scenario: accepted records suddenly fall
Compare input/accepted/rejected and group rejection reasons. Check schema, nulls, conversion, duplicate rules and recent changes. Identify source vs pipeline cause, communicate impact, correct and replay safely.

## 21. 90-second project explanation
“I built a TechStore order cleaning pipeline. It preserves raw data, profiles row counts/types/nulls/duplicates, and applies explicit rules for required IDs, amount parsing, status normalization and duplicate identity. Rows are accepted or rejected with reasons. I reconcile input with accepted plus rejected, write separate outputs, reload them and run assertions. I also simulate a supplier rename to demonstrate drift detection. The key design choice is not to silently delete or guess values; every decision is traceable.”

## Answer quality checklist
Definition → why it matters → concrete example → risk/decision → proof/test → failure handling.

Avoid vague answers like “I remove nulls” or “I drop duplicates” without a business rule.
