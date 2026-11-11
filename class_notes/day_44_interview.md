# Day 44 Interview Guide — Data Marts and Subject-Oriented Modeling

## 1. What is a data mart?

A data mart is a focused analytical area organized around a business subject such as sales, finance, marketing or inventory. It exposes trusted data for a specific group of users or decisions. It may be implemented as a schema, model set or logical area inside a larger warehouse.

## 2. What does subject-oriented modeling mean?

It means organizing analytical data around business topics and questions rather than copying the source application's table structure. For example, a sales mart combines the relevant order, product, customer, date and channel information for sales analysis.

## 3. What is the difference between a warehouse and a mart?

A warehouse is the broader integrated analytical environment across business domains. A mart is a focused subject area that serves a team or use case. A mart often uses shared warehouse data and definitions.

## 4. What is a dependent data mart?

A dependent mart is built from an integrated enterprise warehouse. It can reuse shared dimensions, keys and business definitions, making cross-domain consistency easier.

## 5. What is an independent data mart?

An independent mart is built directly from source systems rather than from a shared enterprise warehouse. It may deliver an isolated requirement quickly but can duplicate ingestion and business logic and create conflicting definitions.

## 6. What is a conformed dimension?

A conformed dimension is shared consistently by multiple facts or marts. For example, sales and inventory models may use the same product dimension and category definitions. The business meaning and key mapping must truly be shared.

## 7. What is a semantic layer?

A semantic layer presents business-friendly concepts and governed metric definitions over underlying data models. It helps users reuse consistent calculations such as paid sales, active customers and refund rate rather than recreating them differently in each report.

## 8. What is a metric contract?

A metric contract documents a metric's name, business definition, grain, formula, filters, date role, owner, refresh expectation and known limitations. It prevents different reports from using the same label for different calculations.

## 9. Why should business questions come before table design?

The questions reveal the required metrics, dimensions, time roles, users and decisions. Designing tables first can produce a model that is technically valid but cannot answer the real business question or uses the wrong grain.

## 10. Why might sales and refunds use separate fact tables?

They represent different business processes and may have different grains. One order line can have multiple refund events. Joining them carelessly can multiply sales rows and overstate totals. Separate facts can share dimensions and be reconciled using clear business rules.

## 11. Why is COUNT(*) not always the order count?

A sales fact may have one row per order line. An order with three lines contributes three rows. To count orders, use a distinct order identifier or a model whose grain is one row per order.

## 12. How do you design a sales mart?

I clarify the users and decisions, define sales and refund metrics, choose the reporting date, identify source systems, declare fact grain, select dimensions, reuse conformed dimensions, build presentation models, validate totals and document ownership and freshness.

## 13. What is mart ownership?

Business ownership defines the meaning and acceptable use of the data. Technical ownership maintains the models, pipelines, tests, documentation and operational behavior. Both are important.

## 14. Scenario: two departments show different sales totals.

I compare metric definitions, grain, date role, filters, refunds, cancellations and currency rules. I trace both reports to their underlying models and identify where the definitions diverge. I would agree one governed definition with the business owner and make reports reuse it.

## 15. Scenario: adding refunds doubles sales.

I would inspect the join cardinality and grain. A sales line may match multiple refund events, multiplying the sales amount. I would keep separate facts or aggregate refunds to a compatible grain before combining them, then reconcile totals.

## 16. Scenario: the CEO needs sales by category and inventory by category.

I would use a conformed product dimension if both processes share the same product identity and category definitions. I would keep sales and inventory as separate facts because their grains and measures differ.

## 17. Explain your Day 44 sales mart.

I translated sales reporting questions into metrics and dimensions, declared the sales fact grain, identified date, product, customer and channel dimensions, separated refund events, and documented the metric definitions, owners, refresh expectations and validation checks. I then built a reporting model and reconciled its totals.

## Interview traps

- Calling every subject-area schema a separate warehouse.
- Building a mart by copying source tables without business definitions.
- Duplicating dimensions that should be conformed.
- Joining facts with incompatible grains.
- Confusing line count with order count.
- Assuming a semantic layer fixes incorrect underlying data.
- Publishing a mart without owner or freshness expectation.

## Readiness checklist

- [ ] Explain warehouse versus mart.
- [ ] Explain dependent versus independent marts.
- [ ] Explain conformed dimensions.
- [ ] Define a metric contract.
- [ ] Defend separate facts for separate processes.
- [ ] Explain the sales mart grain and validation.
