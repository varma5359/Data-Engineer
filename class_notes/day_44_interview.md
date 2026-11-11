# Day 44 Interview Guide — Data Marts and Subject-Oriented Modeling

**Interview goal:** Turn business questions into a governed analytical product and defend its grain, metrics, relationships and quality.

## 1. What is a data mart?

A data mart is a focused analytical area organized around a subject—sales, finance, inventory or marketing—and designed for a defined group of users or decisions. It can be a schema, a set of warehouse models or a logical presentation area; it does not necessarily require a separate server.

**Example:** A sales mart exposes paid sales, order counts, product categories, customer regions and sales channels without requiring a sales manager to understand every source-system table.

## 2. What is subject-oriented modeling?

It organizes data around business questions rather than mirroring source application structures.

Source-oriented: customer application table, order application table, payment table.  
Subject-oriented: sales performance, customer behavior, payment performance.

The subject drives the metrics, grain and dimensions.

## 3. Warehouse versus mart?

The warehouse is the broad integrated analytical environment across domains. A mart is a focused area serving a subject or use case, often built from warehouse data. A mart should reuse enterprise definitions rather than invent a conflicting revenue calculation.

## 4. Dependent versus independent mart?

A **dependent mart** is built from an integrated warehouse. It can reuse conformed dimensions, keys and governed metrics.

An **independent mart** ingests directly from source systems. It may be quicker for a narrow request but risks duplicated pipelines, inconsistent definitions and isolated data silos.

I would generally prefer dependent marts when an enterprise warehouse and governed models exist.

## 5. What is a conformed dimension?

A dimension whose attributes and business meaning are consistent across facts/marts. Sales and inventory may both use a product dimension.

Conformance is not merely having tables with the same column names. Product identity, category definitions, keys and change handling must be aligned.

## 6. What is a semantic layer?

A semantic layer exposes business-friendly dimensions and governed metrics on top of data models. It can help BI users reuse definitions such as Paid Sales or Active Customer rather than independently recreating them.

It does not repair incorrect data, incorrect grain or bad metric definitions underneath.

## 7. What is a metric contract?

A metric contract specifies:
- metric name and business meaning;
- formula and grain;
- eligible records and exclusions;
- date role;
- currency/unit;
- owner and approval;
- refresh/freshness expectation;
- known limitations and version history.

**Example:** Paid Sales = sum of eligible paid sales-line amounts, after the approved discount treatment, in the reporting currency. Refunds are reported separately unless the business explicitly defines net sales.

## 8. Why define grain before writing SQL?

Grain is what one row represents. If sales fact is one row per order line, an order with three lines contributes three rows. COUNT(*) is line count, not order count.

Grain determines:
- which measures can be summed;
- which keys identify a row;
- how joins behave;
- how counts are calculated;
- how facts can be combined safely.

A grain mismatch can inflate metrics even when SQL runs successfully.

## 9. Why keep sales and refund facts separate?

They represent different events and often have different grains and dates. One sales line can have multiple partial refund events, and refunds may occur after the sale.

Joining raw refund events directly to sales lines can repeat the sales amount. Keep separate facts, share appropriate dimensions, aggregate at compatible grains and define whether refunds are analyzed by refund date or original order date.

## 10. How do you count orders from an order-line fact?

Use COUNT(DISTINCT order_id) for the eligible population, or use a separate order-grain model. COUNT(*) counts sales lines. Confirm whether partially paid, cancelled or split orders qualify.

## 11. How do you design a sales mart from a request?

I would:
1. identify users and decisions;
2. capture reporting questions;
3. agree metric definitions and date roles;
4. declare fact grain;
5. identify fact processes and dimensions;
6. reuse conformed dimensions;
7. build a presentation model/view;
8. validate keys, row counts and metric totals;
9. document ownership, freshness, access and limitations.

## 12. How do you validate a mart?

- Reconcile eligible source and mart totals with identical filters.
- Check fact grain and distinct business keys.
- Check dimension key uniqueness and unmatched relationships.
- Compare row counts before/after joins to detect fan-out.
- Verify metric definitions and date roles.
- Confirm source completeness and refresh freshness.
- Test representative business questions against known examples.

Equal totals alone are not sufficient: two errors can cancel each other.

## 13. Scenario: Finance and Sales report different revenue.

I compare their definitions, grain, filters, date roles, discounts, cancellations, refunds, currency and source populations. I trace each report to its model, quantify the difference, and work with metric owners to agree one governed definition. Then I update reusable models/tests and communicate the change rather than silently changing one dashboard.

## 14. Scenario: joining refunds doubled sales.

A sale line probably matched multiple refund events, multiplying the sale amount. I inspect join cardinality and row counts, keep the facts separate or aggregate refund events to a compatible grain, and reconcile sales and refund totals independently.

## 15. Scenario: CEO wants sales and inventory by category.

Use a conformed product/category dimension if identity and category definitions are shared. Keep sales and inventory facts separate: sales may be order-line grain; inventory may be product-location-snapshot grain. Do not directly join raw facts at incompatible grains and sum both measures.

## 16. Scenario: source is missing but mart refresh succeeded.

A successful SQL job is not proof of complete data. I check expected source delivery, source row counts and freshness. If a critical source is absent, the quality gate should block or clearly mark the mart as incomplete according to the agreed SLA.

## 17. Explain your Day 44 Sales Mart project.

"I translated sales questions into a metric contract, defined sales-line and refund-event grains, selected relevant dimensions and reused warehouse keys. I built a reporting view, validated totals with identical eligibility filters, checked dimension uniqueness and join fan-out, and documented metric ownership, refresh and limitations."

**Evidence:** metric contract, grain statements, mart diagram, SQL output, source-to-mart reconciliation, key checks and documented refund treatment.

## 18. Interview traps

- Building tables before understanding decisions.
- Treating a mart as a copy of source tables.
- Using COUNT(*) for orders on a line-grain fact.
- Joining facts at incompatible grains.
- Assuming a semantic layer guarantees correctness.
- Comparing totals with different filters or date roles.
- Publishing without freshness, owner or metric definition.

## 19. Practice questions with answers

**Q1.** An order has four sales lines. What does COUNT(*) return for that order's lines?  
**Answer:** Four. Order count requires distinct order IDs or order-grain data.

**Q2.** Can sales and refunds be summed after a raw join?  
**Answer:** Not safely; multiple refunds can multiply sale rows. Aggregate separately at compatible grain.

**Q3.** What makes a product dimension conformed?  
**Answer:** Shared identity, meaning, keys and governed attributes—not just similar columns.

**Q4.** Why can two reports show different sales?  
**Answer:** Different definitions, populations, date roles, filters, currency or grain.

**Q5.** What does equal source/mart total prove?  
**Answer:** It is one reconciliation signal, not complete correctness; validate grain, keys, filters and representative records too.

## Readiness checklist

- [ ] Explain warehouse, mart and semantic layer.
- [ ] Define a metric contract and grain.
- [ ] Defend sales/refund fact separation.
- [ ] Explain conformed dimensions.
- [ ] Diagnose fan-out and order-count errors.
- [ ] Present mart validation evidence and ownership.
