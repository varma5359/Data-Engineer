# Day 26: Database Normalization
**Phase:** 3 — SQL and Database Engineering  
**Mini-project:** Data Model Comparison  
**Learning time:** 2–3 hours  
**Audience:** Beginners progressing to junior Data Engineer level  
**Database:** PostgreSQL for practical table examples.

---

## 1. Why do we need normalization?

Imagine TechStore stores every order in one large spreadsheet:

| order_id | customer_name | customer_email | product_name | product_price | quantity |
|---:|---|---|---|---:|---:|
| 5001 | Asha | asha@example.com | Keyboard | 1200 | 1 |
| 5001 | Asha | asha@example.com | Mouse | 700 | 2 |
| 5002 | Ravi | ravi@example.com | Monitor | 8500 | 1 |
| 5003 | Asha | asha@example.com | Laptop Stand | 1500 | 2 |

The same customer information is repeated for every order line. If Asha changes her email address, we must update every row where Asha appears.

What if one row is updated and another is forgotten? The database now contains conflicting customer information.

Normalization is a database design process that organizes data into related tables to reduce unnecessary duplication and prevent data anomalies.

### Learning outcomes

By the end, you can:
1. Explain data redundancy.
2. Explain insert, update and delete anomalies.
3. Understand functional dependencies at a beginner level.
4. Explain first normal form (1NF).
5. Explain second normal form (2NF).
6. Explain third normal form (3NF).
7. Identify candidate keys and composite keys.
8. Decompose a combined table into related tables.
9. Explain foreign keys after decomposition.
10. Understand when denormalization may be useful.
11. Compare normalized operational design with reporting models.

---

## 2. Understand data redundancy

Redundancy means storing the same fact repeatedly.

In the combined TechStore table:
- Asha's name is repeated.
- Asha's email is repeated.
- The product name and price are repeated for each line where that product is ordered.
- An order's date and customer may be repeated for every item in that order.

Not all repeated values are wrong. Quantity and unit price belong to an order item, so those facts are recorded at the line grain. The problem is repeating facts that belong to a different entity.

A useful design question is:

**Which real-world thing does this fact describe?**

- customer_name describes a customer.
- product_name describes a product.
- order_date describes an order.
- quantity describes an order item.

Store each fact in the table that represents the entity it describes.

---

## 3. Three data anomalies

Poorly organized tables can create anomalies.

### 3.1 Update anomaly

Asha's email is stored in three rows. If she changes it, all three copies must be updated.

If only two rows are changed, the table contains two different emails for the same customer.

### 3.2 Insert anomaly

Suppose TechStore adds a new product but nobody has ordered it yet.

In a combined order-line table, the product may not be insertable without inventing an order. The design makes it difficult to store a product independently.

### 3.3 Delete anomaly

Suppose the only order containing Monitor is deleted. If the product's only stored description is in the same combined table, deleting that order line may also remove the only record of the product.

These problems happen because multiple independent facts have been mixed into one table.

---

## 4. Understand keys before normalization

A key identifies rows and expresses relationships.

### 4.1 Primary key

A primary key uniquely identifies each row and cannot be NULL.

Examples:
- customer_id in customers.
- product_id in products.
- order_id in orders.

### 4.2 Composite key

A composite key uses more than one column to identify a row.

For an order-item design, order_id plus line_number can identify a line within an order. The TechStore training schema uses a separate order_item_id surrogate key, but a composite key is also a common design option.

### 4.3 Candidate key

A candidate key is a minimal set of column(s) that can uniquely identify a row. One candidate key is selected as the primary key. Other candidate keys are alternate keys.

### 4.4 Foreign key

A foreign key refers to a key in another table. It links related records and helps enforce referential integrity.

---

## 5. Functional dependency: which fact determines another?

A functional dependency means one attribute or set of attributes determines another attribute.

Notation:

`customer_id -> customer_name, customer_email`

Read it as: "A customer_id determines that customer's name and email."

If customer_id is 1, the customer name and email should be the same wherever that customer appears.

Another example:

`product_id -> product_name, product_price`

A product ID identifies the product facts.

For an order line, a combination such as order_id and product_id may determine quantity, depending on the business rule that a product appears at most once per order. If the same product can appear on multiple lines, use a line identifier instead.

Functional dependencies help decide which facts belong together.

---

## 6. First Normal Form (1NF)

A table is in first normal form when each cell contains a single value from the relevant domain and rows can be identified uniquely. Repeating groups and arrays of values in one field are avoided in the simple relational model taught here.

### 6.1 A table that violates 1NF

| order_id | customer_name | products |
|---:|---|---|
| 5001 | Asha | Keyboard, Mouse |
| 5002 | Ravi | Monitor |

The products column contains multiple product values in one cell. It is difficult to filter, count and join individual products reliably.

### 6.2 Convert it to 1NF

| order_id | customer_name | product_name |
|---:|---|---|
| 5001 | Asha | Keyboard |
| 5001 | Asha | Mouse |
| 5002 | Ravi | Monitor |

Now each row represents one order-product line and each cell contains one value.

### 6.3 Why this is better

We can:
- Filter product_name = 'Mouse'.
- Count product lines.
- Join a product identifier to the products table.
- Store quantity for each line.

But 1NF alone does not remove all duplication. Asha's details may still repeat.

---

## 7. Second Normal Form (2NF)

A table is in 2NF when:
1. It is in 1NF.
2. Every non-key attribute depends on the whole candidate key, not only part of a composite candidate key.

2NF is especially relevant when a table has a composite key.

### 7.1 Understand partial dependency

Imagine a table:

| order_id | product_id | order_date | product_name | quantity |
|---:|---:|---|---|---:|
| 5001 | 101 | 2026-11-01 | Keyboard | 1 |
| 5001 | 102 | 2026-11-01 | Mouse | 2 |
| 5002 | 103 | 2026-11-02 | Monitor | 1 |

Assume the composite key is (order_id, product_id).

Dependencies:
- order_id determines order_date.
- product_id determines product_name.
- (order_id, product_id) determines quantity.

order_date depends only on part of the composite key (order_id). product_name depends only on part (product_id). These are partial dependencies.

### 7.2 Decompose the table

Orders:

| order_id | order_date |
|---:|---|
| 5001 | 2026-11-01 |
| 5002 | 2026-11-02 |

Products:

| product_id | product_name |
|---:|---|
| 101 | Keyboard |
| 102 | Mouse |
| 103 | Monitor |

Order items:

| order_id | product_id | quantity |
|---:|---:|---:|
| 5001 | 101 | 1 |
| 5001 | 102 | 2 |
| 5002 | 103 | 1 |

Now order_date belongs to an order, product_name belongs to a product, and quantity belongs to the order-product combination.

### 7.3 Important clarification

If a table has only a single-column candidate key, it cannot have a partial dependency on part of that key. 2NF is automatically satisfied if it is already in 1NF and every candidate key is a single attribute.

---

## 8. Third Normal Form (3NF)

A table is in 3NF when:
1. It is in 2NF.
2. Non-key attributes do not depend transitively on a key through another non-key attribute, under the standard beginner explanation.

### 8.1 Understand transitive dependency

Imagine a customer table:

| customer_id | customer_name | city_id | city_name |
|---:|---|---:|---|
| 1 | Asha | 10 | Hyderabad |
| 2 | Ravi | 20 | Chennai |
| 3 | Meena | 30 | Bengaluru |

Dependencies:
- customer_id determines city_id.
- city_id determines city_name.

Therefore customer_id determines city_name through city_id. This is a transitive dependency.

If Hyderabad changes its official display name or city metadata, we would have to update every customer row in that city.

### 8.2 Decompose into Customers and Cities

Customers:

| customer_id | customer_name | city_id |
|---:|---|---:|
| 1 | Asha | 10 |
| 2 | Ravi | 20 |
| 3 | Meena | 30 |

Cities:

| city_id | city_name |
|---:|---|
| 10 | Hyderabad |
| 20 | Chennai |
| 30 | Bengaluru |

Now city_name is stored once in the Cities table.

### 8.3 How do we retrieve the city name?

```sql
SELECT
    c.customer_name,
    ci.city_name
FROM customers_normalized AS c
JOIN cities AS ci
  ON c.city_id = ci.city_id;
```

The JOIN reconstructs the information when a report needs it.

Normalization does not mean information is lost. It means facts are stored in the appropriate related tables.

---

## 9. Compare 1NF, 2NF and 3NF

| Normal form | Main beginner rule | Problem addressed |
|---|---|---|
| 1NF | One value per cell; no repeating groups | Multi-valued cells |
| 2NF | No partial dependency on part of a composite candidate key | Facts depending on only part of a key |
| 3NF | No transitive dependency of non-key attributes on a key | Non-key facts depending on other non-key facts |

Learn the order:
1NF -> 2NF -> 3NF.

A table cannot satisfy 3NF if it has not first satisfied the earlier normal forms.

---

## 10. Build a normalized TechStore model

We will use these tables:

### Customers

| Column | Meaning |
|---|---|
| customer_id | Unique customer identifier |
| customer_name | Customer's name |
| email | Customer email |
| city | Customer city |

### Products

| Column | Meaning |
|---|---|
| product_id | Unique product identifier |
| product_name | Product name |
| category | Product category |
| price | Current catalogue price |

### Orders

| Column | Meaning |
|---|---|
| order_id | Unique order identifier |
| customer_id | Customer who placed the order |
| order_date | Date of order |
| order_status | Current order status |

### Order items

| Column | Meaning |
|---|---|
| order_item_id | Unique order line identifier |
| order_id | Parent order |
| product_id | Product on the line |
| quantity | Number of units |
| unit_price | Price charged per unit on that order line |

Why keep unit_price in order_items if products already has price?

Because product price can change over time. The order line must preserve the price charged at the time of the purchase. This is a historical transaction fact, not simply duplicated catalogue data.

---

## 11. Create a normalized design in PostgreSQL

The Day 20 training database already has the TechStore tables. For this lesson, create a separate schema to demonstrate normalization without changing the existing training tables.

```sql
CREATE SCHEMA IF NOT EXISTS normalization_lab;
```

Create customers:

```sql
CREATE TABLE normalization_lab.customers (
    customer_id INTEGER PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    email VARCHAR(200) NOT NULL UNIQUE,
    city VARCHAR(80)
);
```

Create products:

```sql
CREATE TABLE normalization_lab.products (
    product_id INTEGER PRIMARY KEY,
    product_name VARCHAR(120) NOT NULL,
    category VARCHAR(80) NOT NULL,
    current_price NUMERIC(12, 2) NOT NULL CHECK (current_price >= 0)
);
```

Create orders:

```sql
CREATE TABLE normalization_lab.orders (
    order_id INTEGER PRIMARY KEY,
    customer_id INTEGER NOT NULL
        REFERENCES normalization_lab.customers(customer_id),
    order_date DATE NOT NULL,
    order_status VARCHAR(30) NOT NULL
);
```

Create order items:

```sql
CREATE TABLE normalization_lab.order_items (
    order_item_id INTEGER PRIMARY KEY,
    order_id INTEGER NOT NULL
        REFERENCES normalization_lab.orders(order_id),
    product_id INTEGER NOT NULL
        REFERENCES normalization_lab.products(product_id),
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    unit_price NUMERIC(12, 2) NOT NULL CHECK (unit_price >= 0)
);
```

Notice:
- Each table has its own primary key.
- Orders references customers.
- Order items references orders and products.
- Quantity must be positive.
- Historical unit_price is stored on the order line.

---

## 12. Insert sample data into the normalized model

Insert customers first because orders reference them.

```sql
INSERT INTO normalization_lab.customers
    (customer_id, customer_name, email, city)
VALUES
    (1, 'Asha', 'asha@example.com', 'Hyderabad'),
    (2, 'Ravi', 'ravi@example.com', 'Chennai'),
    (3, 'Meena', 'meena@example.com', 'Bengaluru');
```

Insert products:

```sql
INSERT INTO normalization_lab.products
    (product_id, product_name, category, current_price)
VALUES
    (101, 'Keyboard', 'Accessories', 1200),
    (102, 'Mouse', 'Accessories', 700),
    (103, 'Monitor', 'Electronics', 8500),
    (104, 'Laptop Stand', 'Accessories', 1500);
```

Insert orders:

```sql
INSERT INTO normalization_lab.orders
    (order_id, customer_id, order_date, order_status)
VALUES
    (5001, 1, '2026-11-01', 'PLACED'),
    (5002, 2, '2026-11-02', 'SHIPPED'),
    (5003, 1, '2026-11-03', 'PLACED');
```

Insert order items:

```sql
INSERT INTO normalization_lab.order_items
    (order_item_id, order_id, product_id, quantity, unit_price)
VALUES
    (1, 5001, 101, 1, 1200),
    (2, 5001, 102, 2, 700),
    (3, 5002, 103, 1, 8500),
    (4, 5003, 104, 2, 1500);
```

Verify counts:

```sql
SELECT COUNT(*) FROM normalization_lab.customers;
SELECT COUNT(*) FROM normalization_lab.products;
SELECT COUNT(*) FROM normalization_lab.orders;
SELECT COUNT(*) FROM normalization_lab.order_items;
```

Expected counts: 3, 4, 3 and 4.

If you rerun the inserts, primary key or unique constraint errors are expected because those records already exist. Do not delete the schema just to silence an error unless you understand that DROP SCHEMA ... CASCADE removes all objects in it.

---

## 13. Reconstruct a business report with JOIN

A normalized database stores facts in related tables. A report can join them when it needs a combined view.

```sql
SELECT
    c.customer_name,
    o.order_id,
    o.order_date,
    p.product_name,
    oi.quantity,
    oi.unit_price,
    oi.quantity * oi.unit_price AS line_total
FROM normalization_lab.customers AS c
JOIN normalization_lab.orders AS o
  ON c.customer_id = o.customer_id
JOIN normalization_lab.order_items AS oi
  ON o.order_id = oi.order_id
JOIN normalization_lab.products AS p
  ON oi.product_id = p.product_id
ORDER BY o.order_id, oi.order_item_id;
```

Expected: four order-line rows with line totals 1,200, 1,400, 8,500 and 3,000.

The data is not lost by normalization. The query reconstructs the required view by following foreign-key relationships.

---

## 14. Normalization versus denormalization

Normalization reduces unnecessary duplication in operational systems. But analytical systems may intentionally store pre-joined or repeated values to make reporting simpler or faster.

Denormalization means introducing controlled redundancy for a specific purpose.

Examples:
- A reporting table stores customer city alongside order totals.
- A data warehouse dimension repeats descriptive attributes for easier analytics.
- A materialized summary stores daily sales totals.

Denormalization should be deliberate. You need a reliable refresh or update process so duplicated facts do not drift out of sync.

### 14.1 Operational database

Typical priorities:
- Correct inserts and updates.
- Avoiding update anomalies.
- Referential integrity.
- Transactional consistency.
- Efficient operational lookups.

Normalized models are commonly useful here.

### 14.2 Analytical database

Typical priorities:
- Fast reporting queries.
- Aggregation across large datasets.
- Easy business analysis.
- Stable reporting dimensions.

Dimensional models such as star schemas intentionally denormalize dimension attributes. You will study data warehousing and dimensional modeling in a later phase.

---

## 15. When should we denormalize?

Consider denormalization when:
- A measured query repeatedly performs expensive joins.
- A reporting workload benefits from a precomputed summary.
- A data warehouse needs a star schema for simpler analytics.
- The duplicated data can be refreshed and reconciled reliably.

Do not denormalize just because joins look complicated. First understand the data model, workload and correctness requirements.

---

## 16. Common beginner mistakes

1. Thinking normalization means splitting every column into a separate table.
2. Thinking 1NF means every value must be unique across the table.
3. Forgetting that 2NF concerns partial dependencies on a composite candidate key.
4. Confusing partial dependency with transitive dependency.
5. Thinking a foreign key must be unique.
6. Removing historical transaction values because a current product price exists.
7. Believing normalization removes the need for JOINs.
8. Assuming denormalization is always bad.
9. Duplicating data without a refresh strategy.
10. Treating a table with a surrogate primary key as automatically normalized.
11. Forgetting to identify the row grain before splitting tables.
12. Assuming 3NF guarantees every possible data-quality rule.

---

## 17. Mini-project: Data Model Comparison

### Business requirement

Compare a combined order spreadsheet with a normalized relational model.

### Task 1: identify repeated facts

In the combined design, mark:
- Customer name and email.
- Product name and catalogue price.
- Order date.
- Order-item quantity and charged unit price.

Identify which entity each fact describes.

### Task 2: identify anomalies

Write one example each of:
- Update anomaly.
- Insert anomaly.
- Delete anomaly.

### Task 3: identify dependencies

Document:
- customer_id -> customer_name, email, city
- product_id -> product_name, category, current_price
- order_id -> customer_id, order_date, order_status
- order_item_id -> order_id, product_id, quantity, unit_price

### Task 4: explain normal forms

Use the product list in one cell as a 1NF problem. Use the composite order_id/product_id table to demonstrate partial dependencies for 2NF. Use customer_id -> city_id -> city_name to demonstrate a transitive dependency for 3NF.

### Task 5: create the normalized schema

Run the CREATE TABLE statements in normalization_lab.

### Task 6: insert the sample records

Insert customers, products, orders and order items in parent-before-child order.

### Task 7: reconstruct the report

Run the multi-table JOIN query and verify four rows.

### Task 8: compare the models

| Topic | Combined table | Normalized tables |
|---|---|---|
| Customer email update | Multiple rows may need changes | Update one customer row |
| Add product before first sale | Awkward if order data is required | Insert product independently |
| Delete last order containing product | May lose product details | Product remains in products |
| Reporting | Fewer joins initially | JOINs required to combine facts |
| Redundancy | Higher | Reduced for entity facts |

### Project checklist

- [ ] I can explain the three anomalies.
- [ ] I can identify a candidate/composite key.
- [ ] I can explain a functional dependency.
- [ ] I can explain 1NF with a multi-valued cell.
- [ ] I can explain 2NF with a partial dependency.
- [ ] I can explain 3NF with a transitive dependency.
- [ ] I created the normalized tables.
- [ ] I inserted parent and child records in the correct order.
- [ ] I reconstructed the combined report with JOINs.
- [ ] I can explain why order_items stores historical unit_price.
- [ ] I can explain when denormalization is useful.

---

## 18. Practice questions

1. What is normalization?
2. What is data redundancy?
3. Explain update anomaly with a customer email.
4. Explain insert anomaly with a new product.
5. Explain delete anomaly with a product's only order.
6. What is a functional dependency?
7. What is a candidate key?
8. What is a composite key?
9. Explain 1NF.
10. Explain 2NF.
11. Explain partial dependency.
12. Explain 3NF.
13. Explain transitive dependency.
14. Why does a normalized model need JOINs?
15. Why store unit_price in order_items when products has current_price?
16. What is denormalization?
17. Give a reporting example where denormalization is useful.
18. What risk does denormalization introduce?

### Answer hints

1. Organizing related facts into tables to reduce unnecessary redundancy and anomalies.
2. Repeated storage of the same fact.
3. One copy changes and another is missed.
4. Cannot store an independent product without inventing an order.
5. Removing an order line removes the only product record.
6. One attribute or set of attributes determines another.
7. A minimal attribute set that uniquely identifies a row.
8. A key made from multiple columns.
9. Atomic values and no repeating groups in the simple relational model.
10. 1NF plus no partial dependency on part of a composite candidate key.
11. A non-key fact depends on only part of a composite key.
12. 2NF plus no transitive non-key dependency in the standard beginner model.
13. A key determines a non-key attribute through another non-key attribute.
14. Facts are separated into entity tables and joined when a combined view is needed.
15. It preserves the price actually charged at purchase time.
16. Controlled redundancy for performance or usability.
17. A star-schema dimension or reporting summary.
18. Duplicated facts can become inconsistent if refresh/update logic fails.

---

## 19. Day 26 revision

- Normalization organizes facts into related tables.
- Redundancy can create update, insert and delete anomalies.
- Functional dependencies help identify where facts belong.
- 1NF removes repeating groups and multi-valued cells in the simple relational model.
- 2NF removes partial dependencies on part of a composite candidate key.
- 3NF removes transitive non-key dependencies in the standard beginner explanation.
- Primary keys identify rows.
- Foreign keys connect related tables.
- Normalization reduces unnecessary duplication but does not remove the need for JOINs.
- Denormalization introduces controlled redundancy for a purpose.
- Historical facts such as order-time price should not be replaced by current catalogue price.

**You are ready for Day 27 when you can explain 1NF, 2NF and 3NF using the TechStore examples and reconstruct the normalized data with JOINs.**
