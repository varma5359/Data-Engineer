# Day 26: Database Normalization

**Pathway:** TechPhile.ai — Data Engineer Career Pathway  
**Phase:** 3 — SQL and Database Engineering  
**Audience:** Freshers, college students and junior Data Engineer candidates  
**Level:** Beginner to junior | **Class time:** 2–3 hours  
**Approved syllabus:** 1NF, 2NF, 3NF, normalization, denormalization, data redundancy, database design trade-offs  
**Mini-project:** Data Model Comparison  
**Primary database:** PostgreSQL; MySQL differences are identified where relevant.

---

## 1. How to study

Read one concept, predict what the query or program will do, execute the small example, compare the result and explain it aloud. Do not run the entire lesson as one large script.

## 2. Prerequisite and learning outcomes

Use the TechStore training database from Day 20: customers, products, orders and order_items. Confirm you are connected to the correct practice database. If sample data differs, inspect it before comparing outputs. Never experiment on production data.

By the end, you should understand each item in today's syllabus, execute the main examples, explain the output, recognize common mistakes and complete **Data Model Comparison**.

## 3. Why normalize?

A spreadsheet may repeat customer name/email/city and product name/price for every order item. Updating only some copies creates inconsistent facts. Normalization organizes facts to reduce unnecessary duplication and prevent anomalies.

## 4. Anomalies

Example combined order export:

| order_id | customer | email | product_id | product | qty |
|---:|---|---|---:|---|---:|
| 5001 | Asha | asha@example.com | 101 | Keyboard | 1 |
| 5001 | Asha | asha@example.com | 102 | Mouse | 2 |
| 5002 | Ravi | ravi@example.com | 103 | Monitor | 1 |

- Update anomaly: one copy of email changes, another does not.
- Insert anomaly: cannot add a product before it has an order.
- Delete anomaly: deleting the only order can delete the only product description.

## 5. First Normal Form (1NF)

1NF requires atomic values at the chosen row meaning and no repeating groups. Do not store "Keyboard, Mouse" in one cell when products must be queried individually. Store one order-item per row.

Atomicity depends on use: address can be one value in one application, or split into street/city/postcode when each part must be queried separately.

## 6. Second Normal Form (2NF)

A table is in 2NF when it is in 1NF and every non-key attribute depends on the whole candidate key, not only part of a composite key.

If an order-line key is (order_id,product_id), order_date depends only on order_id and product_name only on product_id. These are partial dependencies. Separate:
- orders(order_id,order_date,customer_id)
- products(product_id,product_name)
- order_items(order_id,product_id,quantity)

A single-column key cannot have a partial dependency on part of that key. 2NF matters especially with composite candidate keys.

## 7. Third Normal Form (3NF)

Beginner intuition: 3NF is 2NF plus non-key attributes do not depend transitively on the key through another non-key attribute.

If customer_id determines city_id and city_id determines city_name, separate cities(city_id,city_name) from customers(customer_id,customer_name,city_id). Formal 3NF is defined using functional dependencies and candidate keys; this is an introductory rule of thumb.

## 8. Functional dependencies

X → Y means X determines one Y value under business rules. Examples: customer_id → customer_name/email/city; product_id → product_name/category/current price; order_id → customer_id/date/status. Dependencies help identify candidate keys and decide which facts belong together.

## 9. TechStore normalized model

customers(customer_id,name,email,city); products(product_id,name,category,price); orders(order_id,customer_id,date,status); order_items(order_item_id,order_id,product_id,quantity,unit_price).

unit_price belongs on order_items because it records historical price charged; current catalog price may change.

## 10. Normalization and denormalization

Normalized OLTP reduces repeated facts and supports reliable writes but may require more joins. Denormalized analytics intentionally repeats/precomputes facts for read/reporting performance, but needs consistency and refresh strategy. Warehouses often use star schemas. Neither design is always best; choose based on workload and consistency needs.

## 11. Mini-project — Data Model Comparison

Create Version A combined order export and Version B normalized four-table design. Identify candidate keys/dependencies, show 1NF/partial/transitive dependencies and anomalies, explain benefit/cost of normalization and why a warehouse may use a denormalized star schema.

**Deliverables:** table maps/ERDs, dependency list, anomaly examples, comparison document.

## 12. Self-check

Explain 1NF, 2NF, 3NF, functional dependency, candidate key, anomalies, denormalization and why order_items stores unit_price.

## 99. Next lesson

Next: Day 27 — Database Objects and Automation.