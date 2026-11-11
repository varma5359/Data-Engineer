# Day 47 Interview Guide — Metadata and Data Documentation

## 1. What is metadata?

Metadata is information that describes data. It can describe structure, business meaning, processing history, ownership, quality, classification and access. For example, a sales amount column needs a type, currency, definition, grain, owner and source.

## 2. What are the main metadata categories?

Technical metadata describes schemas, columns, data types, keys and dependencies. Business metadata describes terms, definitions, metrics and usage. Operational metadata describes runs, loads, counts, freshness and quality results. Governance metadata describes ownership, sensitivity, access and retention.

## 3. What is a data catalog?

A data catalog is an inventory and discovery system for data assets. It helps users find datasets, understand their meaning, identify owners, inspect lineage and quality, and request access.

## 4. What is a data dictionary?

A data dictionary describes tables and columns, including data types, nullability, definitions, examples, key roles and units. It helps engineers and analysts interpret fields correctly.

## 5. What is a business glossary?

A business glossary contains approved business terms and definitions, such as paid order, net sales or active customer. It helps teams use consistent language and calculations.

## 6. What is the difference between a catalog and a dictionary?

A dictionary focuses on detailed structure and field definitions. A catalog is broader: it can include asset discovery, ownership, classification, lineage, quality and access, as well as dictionary information.

## 7. Why document grain?

Grain explains what one fact row represents. Without it, users may count rows as orders when rows actually represent order lines, causing incorrect metrics.

## 8. What is a data owner?

A data owner is accountable for the business meaning, acceptable use and rules of a dataset. Technical teams may maintain the implementation, but the business owner approves definitions that affect business interpretation.

## 9. What is a data steward?

A data steward helps maintain definitions, quality expectations, classification and governance practices. Responsibilities vary by organization and may overlap with owner roles.

## 10. What is a data contract?

A data contract is an agreement between producer and consumer about schema, field meaning, keys, required values, delivery behavior, quality expectations and change management.

## 11. Why classify data?

Classification determines how data may be accessed, shared, stored and retained. Sensitive personal or financial information may require restricted access, masking and special handling.

## 12. What is operational metadata?

It describes how a dataset was processed: run ID, source batch, load timestamp, watermark, row counts, rejected records, duration, status and quality results. It supports troubleshooting and freshness assessment.

## 13. What does information_schema provide?

In PostgreSQL it can expose structural metadata such as columns, data types and nullability for accessible database objects. It does not automatically provide business definitions, owners or approved metric logic.

## 14. Scenario: a column is named amt. How do you document it?

I would identify what amount it represents, currency, precision, grain, source, null behavior, transformation, business owner and sensitivity. I would rename it to a clear name where appropriate and update downstream documentation.

## 15. Scenario: the source changes a field type.

I would inspect the schema change, confirm whether it is compatible, check leading zeros or formatting semantics, identify downstream dependencies through lineage, agree the migration with the owner, update the contract and tests, and communicate the impact.

## 16. Scenario: two teams define active customer differently.

I would document the competing definitions and ask the business owners to approve the intended meaning for each use case. If both definitions are valid, they should have distinct names and explicit rules rather than sharing one ambiguous metric label.

## 17. How do you keep documentation current?

Include documentation changes in the model/schema change process. Review owner, definitions, lineage, classification and refresh metadata whenever a dataset changes. Automated structural extraction can help, but business meaning still needs owner review.

## 18. Explain your Day 47 documentation project.

I created catalog entries for key warehouse assets, documented column definitions and grain, wrote a business glossary, recorded owners and classifications, traced important columns to sources and inspected PostgreSQL structural metadata. I separated automatically discoverable technical metadata from business definitions that require human ownership.

## Interview traps

- Calling column names a complete dictionary.
- Treating information_schema as a full business catalog.
- Omitting units and currency.
- Omitting grain.
- Publishing sensitive examples.
- Assuming documentation stays correct without change management.

## Readiness checklist

- [ ] Explain metadata categories.
- [ ] Compare catalog, dictionary and glossary.
- [ ] Explain owner and steward.
- [ ] Describe a data contract.
- [ ] Explain classification and access.
- [ ] Defend the documentation artifacts.
