# Day 26 Interview Preparation: Database Normalization

**Phase:** 3 — SQL and Database Engineering  
**Level:** Beginner to junior Data Engineer  
**Mini-project:** Data Model Comparison

---

## 1. Core questions with complete answers

### 1. What is database normalization?

Normalization is a database design process that organizes facts into related tables to reduce unnecessary duplication and prevent insert, update and delete anomalies. It uses keys and functional dependencies to decide where each fact belongs.

### 2. Why do we normalize a database?

We normalize to reduce inconsistent repeated facts, make updates safer, support independent entity creation and deletion, and improve data integrity. Normalization does not mean avoiding every repeated value; it means storing each fact at the correct grain.

### 3. What is data redundancy?

Data redundancy is repeated storage of the same fact. For example, storing the same customer's email on every order line repeats a customer fact that should normally belong to the customer record.

### 4. What is an update anomaly?

An update anomaly occurs when one fact is stored in several rows and only some copies are updated. The database then contains conflicting values, such as two different emails for the same customer.

### 5. What is an insert anomaly?

An insert anomaly occurs when the table design prevents an entity from being stored independently without inventing unrelated data. For example, a combined order-line table may make it difficult to add a product before it has been ordered.

### 6. What is a delete anomaly?

A delete anomaly occurs when deleting one record unintentionally removes another fact. If a product's only description is stored in its last order line, deleting that line may remove the only record of the product.

### 7. What is a functional dependency?

A functional dependency means one attribute or set of attributes determines another attribute. For example, customer_id -> customer_name, email means a customer ID identifies the customer's name and email.

### 8. What is a candidate key?

A candidate key is a minimal set of attributes that uniquely identifies a row. One candidate key is selected as the primary key; other candidate keys are alternate keys.

### 9. What is a composite key?

A composite key uses more than one column to identify a row. For example, (order_id, line_number) can identify a line within an order if line_number is unique within each order.

### 10. What is 1NF?

In the simple relational model taught here, first normal form requires each cell to hold one atomic value from its domain and avoids repeating groups or multi-valued cells. A comma-separated list of product names in one column is a common violation.

### 11. Does 1NF mean every value in a column must be unique?

No. A city column can contain Hyderabad for many customers. 1NF concerns atomic values and row structure, not uniqueness of every value. Keys provide uniqueness rules.

### 12. What is 2NF?

A table is in 2NF when it is in 1NF and every non-key attribute depends on the whole of every candidate key, not only part of a composite key. It is especially relevant when candidate keys contain multiple columns.

### 13. What is a partial dependency?

A partial dependency occurs when a non-key attribute depends on only part of a composite candidate key. If (order_id, product_id) is the key but order_date depends only on order_id, order_date has a partial dependency.

### 14. Does a single-column primary key automatically satisfy 2NF?

If the table is already in 1NF and all candidate keys are single-column, there is no proper subset of a candidate key on which a partial dependency can exist. So 2NF is satisfied.

### 15. What is 3NF?

In the standard beginner explanation, a table is in 3NF when it is in 2NF and non-key attributes do not depend transitively on a key through another non-key attribute. Formal definitions use candidate keys and prime attributes, but the practical goal is to keep independent facts in their proper tables.

### 16. What is a transitive dependency?

A transitive dependency occurs when a key determines a non-key attribute through another non-key attribute. For example, customer_id determines city_id, and city_id determines city_name. Storing city_name in every customer row repeats a city fact.

### 17. 2NF versus 3NF?

2NF addresses partial dependencies on part of a composite candidate key. 3NF addresses transitive dependencies of non-key facts through other non-key facts. Both reduce anomalies, but they solve different dependency problems.

### 18. Does normalization remove all duplication?

No. It reduces unnecessary duplication of facts. Repeated foreign keys are necessary to represent relationships. Transaction facts may also legitimately repeat values such as the price charged on different order lines.

### 19. Why do normalized databases need JOINs?

Related facts are stored in separate entity tables. A JOIN reconstructs the combined view when a report needs customer, order and product information together. The separation protects update consistency.

### 20. Why is unit_price stored in order_items if products already has price?

products.current_price represents the current catalogue price. order_items.unit_price represents the price actually charged at the time of the order. Product prices can change, so the historical transaction value must be preserved.

### 21. What is denormalization?

Denormalization introduces controlled redundancy to improve reporting simplicity or performance. Examples include a precomputed daily sales summary or a star-schema dimension that stores descriptive attributes together.

### 22. Is denormalization bad?

No. It is a design trade-off. It can improve analytical query performance and usability, but duplicated facts need a reliable refresh and reconciliation strategy to prevent inconsistencies.

### 23. What is the difference between OLTP and analytical design?

Operational OLTP systems prioritize reliable transactions, updates and integrity; normalized designs are commonly useful. Analytical systems prioritize reporting and aggregation; dimensional models often denormalize dimensions for simpler queries. Actual design depends on workload.

### 24. What is a surrogate key?

A surrogate key is a generated identifier with no direct business meaning, such as an integer customer_id. It provides a stable key even if business attributes such as email or name change.

### 25. What is a natural key?

A natural key is an attribute with business meaning that can identify a record, such as a government-issued identifier or a carefully governed product code. Natural keys can change or have privacy constraints, so they need careful selection.

### 26. Can a foreign key contain duplicate values?

Yes. A foreign key commonly repeats in a one-to-many relationship. Many orders can reference the same customer_id. A foreign key does not have to be unique unless an additional uniqueness rule is declared.

### 27. How do you identify the correct table for a column?

Ask what real-world entity or event the fact describes and what key determines it. Customer email belongs to customer; order_date belongs to order; quantity belongs to an order item; the charged unit price belongs to the transaction line.

### 28. What are the limitations of normalization?

A highly normalized model may require more joins for reporting, and complex analytical queries can become harder to write. Normalization also does not automatically solve all business-rule or data-quality problems.

### 29. When would you denormalize?

When measured reporting workloads benefit from a precomputed or repeated representation, such as a summary table or dimensional model, and the team can maintain consistency through controlled refreshes.

### 30. Does adding a surrogate primary key guarantee 3NF?

No. A surrogate key guarantees a convenient row identifier, but the table may still contain partial or transitive dependencies among non-key attributes. Normalization depends on functional dependencies, not merely the existence of an ID column.

---

## 2. Scenario-based questions

### Scenario 1: Customer email is repeated on every order line

**Good answer:** Customer email is a customer fact. I would store it once in customers and reference customer_id from orders. This avoids update anomalies when the email changes.

### Scenario 2: A new product must be added before its first sale

**Good answer:** Product details should be stored independently in a products table. Order items reference product_id only when the product is purchased. This avoids an insert anomaly.

### Scenario 3: Deleting the last order line removes product details

**Good answer:** Product master data is mixed with transactional order data. I would separate products from order_items so deleting a transaction does not delete the product record.

### Scenario 4: A product's catalogue price changes

**Good answer:** Update the current product price in products, but retain unit_price on historical order_items. The order line records the actual price charged at purchase time.

### Scenario 5: A report needs customer name, order date and product name

**Good answer:** Store those facts in their appropriate normalized tables and join customers, orders, order_items and products for the report. Normalization does not mean the report cannot combine them.

### Scenario 6: A dashboard query has too many joins

**Good answer:** First measure the workload and understand the query. A reporting layer or star schema may intentionally denormalize data. I would not weaken operational integrity without a clear performance and maintenance plan.

---

## 3. Mini-project defence

Be ready to explain:

1. What was wrong with the combined order spreadsheet?
2. Which update anomaly did you identify?
3. What was the composite candidate key in the 2NF example?
4. Which attributes had partial dependencies?
5. What was the transitive dependency in the customer-city example?
6. Which tables did you create for TechStore?
7. Why does order_items have a foreign key to products?
8. Why is historical unit_price stored on order_items?
9. How did you reconstruct the combined report?
10. When would you choose a denormalized reporting model?

---

## 4. Common interview traps

- Saying every repeated value is a normalization violation.
- Saying 1NF means all column values must be unique.
- Confusing partial dependency with transitive dependency.
- Claiming a surrogate key automatically makes a table normalized.
- Saying foreign keys must be unique.
- Replacing historical transaction price with current product price.
- Saying normalization removes JOINs.
- Saying denormalization is always bad.
- Forgetting that functional dependencies and candidate keys drive normalization.

---

## 5. Readiness checklist

- [ ] I can explain redundancy and all three anomalies.
- [ ] I can explain functional dependencies.
- [ ] I can identify candidate and composite keys.
- [ ] I can explain 1NF, 2NF and 3NF.
- [ ] I can distinguish partial and transitive dependencies.
- [ ] I can design customers, products, orders and order_items.
- [ ] I understand why historical transaction values are preserved.
- [ ] I can explain normalization versus denormalization.
