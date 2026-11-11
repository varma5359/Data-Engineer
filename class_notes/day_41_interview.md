# Day 41 Interview Guide — Fact and Dimension Tables

## 1. What is a fact table?

A fact table stores business events or observations at a defined grain. It commonly contains dimension keys and measures such as quantity, sales amount or duration. For example, a sales fact may contain one row per order line with product, customer and date keys.

## 2. What is a dimension table?

A dimension table stores descriptive context about entities such as customers, products, dates, stores or channels. Analysts use its attributes to filter, group and label facts. A product dimension may contain product name, category and brand.

## 3. What is grain?

Grain is the precise meaning of one row in a fact table. For example, one row per order line means an order with three product lines creates three fact rows. Grain must be declared before measures and keys are designed because it controls how joins and aggregations behave.

## 4. Why is grain important?

If grain is unclear or mixed, measures can be duplicated or aggregated incorrectly. For example, an order-level amount joined to four order-line rows can appear four times. Defining grain prevents this class of modeling error.

## 5. What is a measure?

A measure is a value analyzed from a fact, such as quantity, sales amount, discount or shipping cost. Measures must be interpreted according to their aggregation behavior.

## 6. Explain additive, semi-additive and non-additive measures.

An additive measure can be summed across relevant dimensions; sales amount is a common example. A semi-additive measure can be summed across some dimensions but not all; a daily account balance should not normally be summed across dates. A non-additive measure, such as a percentage or unit price, should not be summed directly.

## 7. What is a Star Schema?

A Star Schema has a central fact table connected directly to dimension tables. It is commonly easy for BI users to understand and often reduces the number of joins needed for reporting. It does not remove the need to define grain and validate relationships.

## 8. What is a Snowflake Schema?

A Snowflake Schema normalizes parts of dimensions into related tables. For example, product can reference category, which references department. This can reduce repeated attributes but adds joins and may make reporting more complex.

## 9. Star versus Snowflake: which is better?

Neither is universally better. A Star Schema is often simpler for analytics and BI consumption. A Snowflake design can help when dimension hierarchies need normalization or controlled reuse. I would choose based on usability, governance, platform performance and maintenance.

## 10. What are the three common fact-table types?

A transaction fact records individual events, such as one order line. A periodic snapshot records state at regular intervals, such as daily inventory by product and warehouse. An accumulating snapshot tracks a process through milestones, such as order created, paid, shipped and delivered.

## 11. What is a degenerate dimension?

A degenerate dimension is a business identifier stored in the fact table without a separate dimension table, such as an order number. It supports transaction identification and drill-through.

## 12. What is a role-playing dimension?

A role-playing dimension is one dimension used for multiple purposes. A date dimension can be referenced by order date, payment date and delivery date keys in the same fact.

## 13. How can a join cause double-counting?

A join can multiply rows when one row matches several rows on the other side. If an order-level amount is repeated for every order line and then summed, the total is overstated. I would inspect grain, key uniqueness and relationship cardinality before changing the query.

## 14. Why should you not solve duplicates with DISTINCT immediately?

DISTINCT can hide the symptom without fixing the cause. The cause may be duplicate source data, an incomplete join condition, a non-unique dimension key or a grain mismatch. I would identify the source of row multiplication and correct the model or join.

## 15. How do you validate a fact table?

I validate the declared grain, business key uniqueness, dimension key relationships, required fields, measure calculations, source-to-target counts and important aggregate totals. I also test joins for unexpected row multiplication.

## 16. Scenario: an order contains several categories. Can an order-grain fact report sales by category?

Not correctly unless the model retains a valid allocation or line-level relationship. If the requirement is category sales, an order-line fact is usually appropriate because each line identifies its product and category.

## 17. Scenario: the business wants average selling price. Should unit prices be summed?

No. Unit price is non-additive. A weighted average can be calculated as total sales amount divided by total quantity, provided both measures follow the same scope and business rules.

## 18. Scenario: a fact row has a product key not found in the dimension.

I would inspect the source product identifier, dimension load timing and key-mapping process. The warehouse may use a controlled unknown or pending member, but the missing relationship must be recorded and resolved. I would not silently discard the fact.

## 19. Explain the Day 41 sales model.

I declared one row per order line, then identified quantity and line sales amount as measures. I created date, product and customer dimensions, linked them to the sales fact with foreign keys, and wrote queries for category, city and month totals. I validated totals and checked for orphan keys and incorrect line calculations.

## 20. What is the most important lesson from dimensional modeling?

Understand the business process and declare the grain before designing the tables. The grain determines what each row means, which measures are valid and how dimensions can be joined without corrupting totals.

## Scenario practice

For each scenario, explain the issue, likely cause, corrective action and validation:

- Revenue doubles after a new join.
- A product category changes.
- A report needs order date and delivery date.
- Inventory is reported daily.
- A report sums percentages.
- A fact arrives before its dimension.

## Readiness checklist

- [ ] Define fact, dimension, measure and grain.
- [ ] Explain Star and Snowflake schemas.
- [ ] Describe transaction, periodic snapshot and accumulating snapshot facts.
- [ ] Explain join fan-out and double-counting.
- [ ] Defend the Day 41 model and its validation evidence.
