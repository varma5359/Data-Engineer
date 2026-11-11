# Day 43 Interview Guide — Keys and Data Relationships

## 1. What is a natural key?

A natural key is an existing business or source identifier, such as a customer number or product SKU. It has meaning outside the warehouse. It can be useful for traceability, but it may change or be unique only within one source system.

## 2. What is a surrogate key?

A surrogate key is an artificial identifier generated and managed by the warehouse. It has no business meaning. It is useful for multi-source integration and for distinguishing historical dimension versions.

## 3. Why not use customer name as a key?

Names can repeat, change or be misspelled. They are descriptive attributes, not reliable identifiers. A stable business key or warehouse surrogate key should identify the row.

## 4. What is a primary key?

A primary key uniquely identifies each row in a table and cannot be NULL. It provides a reliable row identity and can be referenced by foreign keys.

## 5. What is a foreign key?

A foreign key is a column or group of columns that references a key in another table. It represents a relationship and can prevent child rows from referring to missing parent rows when constraints are enforced.

## 6. What is a composite key?

A composite key uses multiple columns together to identify a row. For example, order_id plus line_number may identify one order line when the source guarantees that combination is unique.

## 7. What is a candidate key?

A candidate key is a minimal set of columns that uniquely identifies a row. A table can have several candidate keys. One is selected as the primary key; the others may be enforced as alternate unique keys.

## 8. What is an alternate key?

An alternate key is a candidate key not selected as the primary key. For example, employee_id may be the primary key and company_email may be an alternate key enforced by a unique constraint.

## 9. Why use a surrogate key in a Type 2 dimension?

One business entity can have multiple historical versions. A surrogate key uniquely identifies each version so facts can reference the version valid at the event time.

## 10. Why include source_system with a source key?

Different applications may generate the same identifier. The pair source_system plus source_id distinguishes source identities unless an enterprise identity process has mapped them to one shared entity.

## 11. Explain one-to-many and many-to-many.

One-to-many means one parent row can relate to multiple child rows, such as one customer with many orders. Many-to-many means multiple rows on each side can relate, such as students and courses. A bridge table commonly represents a many-to-many relationship.

## 12. What is a bridge table?

A bridge table stores the relationships between entities in a many-to-many relationship. In dimensional models, it may also need allocation weights or effective dates depending on the business rule.

## 13. What is an orphan record?

An orphan is a child or fact row that references a parent or dimension row that does not exist. Enforced foreign keys can prevent this in relational databases. In analytical platforms, explicit data tests may be needed.

## 14. What is an unknown dimension member?

It is a controlled placeholder dimension row used when a fact arrives before its real dimension record is available or when the key cannot be resolved immediately. The pipeline should track the reason and resolution rather than hiding the issue.

## 15. What is an inferred dimension member?

It is a placeholder dimension record created from limited information when a fact arrives before the full dimension details. The record is completed when the full source data arrives.

## 16. Scenario: CRM customer 501 and Loyalty customer 501 appear. Are they the same customer?

Not automatically. The identifier is only unique within its source unless a global identity contract says otherwise. I would preserve source identity and use an approved matching or enterprise identity process before merging them.

## 17. Scenario: a dimension key is duplicated in a platform without enforced constraints.

I would identify duplicate source mappings, determine the intended row identity, repair the dimension and add an automated uniqueness test. I would also inspect downstream fact references before changing keys.

## 18. Scenario: a fact arrives before its customer.

I would follow the warehouse's late-arriving dimension policy. That may use an unknown member or an inferred dimension member, record the unresolved source identity and later resolve the fact to the correct dimension key. I would validate the correction and reconcile affected facts.

## 19. How do you validate a relationship?

Check uniqueness on the referenced key, required values, orphan rows, expected cardinality and join row counts. Also verify that the relationship matches the business meaning rather than merely matching data types.

## 20. Explain your warehouse key project.

I documented source identities, selected surrogate keys for warehouse dimensions, used a composite key for order lines, created primary and foreign key relationships and tested duplicate and orphan cases. I also documented how unresolved dimension records are handled.

## Interview traps

- Treating a natural key as globally unique without checking source scope.
- Using a person's name as a key.
- Generating a new warehouse key on every reload.
- Forgetting the business key needed for source traceability.
- Joining on only part of a composite key.
- Assuming foreign keys enforce every business rule.
- Hiding unknown members without monitoring.

## Readiness checklist

- [ ] Explain key types with examples.
- [ ] Explain why multi-source identity needs source context.
- [ ] Explain composite keys and relationship cardinality.
- [ ] Describe unknown and inferred members.
- [ ] Demonstrate duplicate and orphan validation.
