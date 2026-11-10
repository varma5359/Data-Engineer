# Day 19 Interview Preparation: Database Fundamentals

**Pathway:** TechPhile.ai — Data Engineer Career Pathway  
**Phase:** 3 — SQL and Database Engineering  
**Level:** Beginner to Junior Data Engineer  
**Syllabus topic:** Database Fundamentals  
**Mini-project:** Operational Database Design

---

## 1. Core database questions

### 1. What is a database?

A database is an organized collection of related data. It allows applications and users to store, retrieve, update and manage information.

**Example:** An e-commerce database stores customers, products, orders and payments.

### 2. What is a DBMS?

A Database Management System is software used to create, manage, secure and access databases.

**Examples:** MySQL, PostgreSQL, SQL Server, Oracle and SQLite.

### 3. What is the difference between a database and a DBMS?

- Database: the organized data.
- DBMS: the software that manages that data.

For example, a retail database can be managed by PostgreSQL.

### 4. What is a relational database?

A relational database organizes data into tables and connects those tables using keys and relationships.

### 5. What is a table?

A table is a structure that stores related records in rows and attributes in columns.

### 6. What is a row?

A row represents one record in a table. One row in Customers represents one customer.

### 7. What is a column?

A column represents an attribute, such as customer_name, email or city.

### 8. What is a database schema?

A schema describes the database structure, including tables, columns, data types, keys and constraints.

### 9. What is an ERD?

An Entity Relationship Diagram visually represents entities, their attributes and the relationships between them.

### 10. What is referential integrity?

Referential integrity ensures that a foreign-key reference points to a valid record in the referenced table, subject to the database's relationship rules.

---

## 2. Primary key and foreign key questions

### 11. What is a primary key?

A primary key uniquely identifies each row in a table. It cannot contain NULL and must not contain duplicate values.

### 12. What is a foreign key?

A foreign key is a column or group of columns that references a key in another table or the same table.

### 13. What is the difference between a primary key and a foreign key?

| Primary key | Foreign key |
|---|---|
| Identifies a row in its own table | References a row in another or the same table |
| Must be unique | Can repeat |
| Cannot be NULL | May be NULL if the relationship is optional |
| One primary-key constraint per table | Multiple foreign-key constraints are possible |

### 14. Can a foreign key contain duplicate values?

Yes. In Customers-to-Orders, many orders can refer to the same customer_id.

### 15. Can a table have multiple foreign keys?

Yes. Order_Items can refer to Orders using order_id and Products using product_id.

### 16. Can a table have a primary key and foreign key together?

Yes. A junction table can have its own primary key and foreign keys to other tables.

### 17. What is a composite key?

A composite key uses more than one column to identify a row.

**Example:** (student_id, course_id) can identify a student's enrollment in a course.

### 18. What is a candidate key?

A candidate key is a minimal column or combination of columns that can uniquely identify a row.

### 19. What is an alternate key?

A candidate key that was not selected as the primary key is called an alternate key.

### 20. What is a natural key?

A natural key uses a meaningful business value, such as an ISBN or a business-assigned product code.

### 21. What is a surrogate key?

A surrogate key is a generated technical identifier, such as customer_id = 101, that has no direct business meaning.

### 22. What is the difference between natural and surrogate keys?

- Natural key: comes from a meaningful real-world or business value.
- Surrogate key: generated for database identification.

Natural values can change or have quality problems, so surrogate keys are often used as stable internal identifiers.

---

## 3. Relationship questions

### 23. What is a one-to-one relationship?

One record in a table is associated with at most one record in another table, and vice versa.

**Example:** Employee and employee identity card, if each employee has one card.

### 24. What is a one-to-many relationship?

One parent record can be associated with many child records.

**Example:** One customer can place many orders.

### 25. What is a many-to-many relationship?

Many records on one side can be associated with many records on the other side.

**Example:** An order can contain many products, and a product can appear in many orders.

### 26. How do you implement a many-to-many relationship in a relational database?

Create a junction table containing foreign keys to both parent tables. Add attributes that describe the relationship when required.

### 27. Why do we need an Order_Items table?

An order can contain multiple products, and each product can appear in many orders. Order_Items resolves this many-to-many relationship and stores quantity and order-time price.

### 28. Where should the foreign key be placed in a one-to-many relationship?

Normally on the many side.

For Customers and Orders, Orders.customer_id references Customers.customer_id.

### 29. Can a customer exist without an order?

Yes, if the business permits customer registration before purchase. The Customers record can exist without a matching Orders record.

### 30. Can an order exist without a customer?

That depends on the business rule. In the TechStore design, every order must belong to an existing customer.

---

## 4. Data types and NULL questions

### 31. Why are data types important?

They define what kind of values a column can store and help the database validate, compare and process values correctly.

### 32. Which data type would you use for a financial amount?

Use an exact decimal type such as DECIMAL or NUMERIC, with suitable precision and scale.

### 33. Why should a postal code often be stored as text?

A postal code is an identifier, not a value used for arithmetic. Text preserves leading zeroes and may support letters in some countries.

### 34. What is the difference between DATE and TIMESTAMP?

DATE stores a calendar date. TIMESTAMP stores a date and time.

### 35. What is NULL?

NULL represents an absent, unknown or not-applicable value. Its meaning should be understood from the business context.

### 36. Is NULL equal to zero?

No. Zero is a numeric value. NULL represents missing or unknown information.

### 37. Is NULL the same as an empty string?

No. An empty string is a text value containing no characters. NULL is an absent value.

---

## 5. Practical database design questions

### 38. Design a database for an online shopping application.

A simple design contains:

- Customers(customer_id, customer_name, email, city)
- Products(product_id, product_name, category, current_price)
- Orders(order_id, customer_id, order_date, order_status)
- Order_Items(order_item_id, order_id, product_id, quantity, unit_price)

Relationships:

- Customers 1:M Orders.
- Orders 1:M Order_Items.
- Products 1:M Order_Items.
- Orders M:N Products through Order_Items.

### 39. Why should customer information not be repeated in every order row?

It creates unnecessary repetition and can cause update inconsistencies. Store customer details once and reference the customer through customer_id.

### 40. Why should product IDs not be stored as comma-separated values?

The database cannot easily validate each value as a foreign key. Searching, updating, joining and aggregating become difficult. Use a junction table.

### 41. Why do we store unit_price in Order_Items when Products already has current_price?

The current product price can change. Order_Items.unit_price preserves the price charged at the time of the order.

### 42. How would you prevent an order from referring to a nonexistent customer?

Define a foreign-key relationship from Orders.customer_id to Customers.customer_id. Validate incoming data before loading as well.

### 43. How would you identify an orphan record in an incoming dataset?

Compare the child records' foreign-key values with the parent dataset. Any child key with no matching parent is an orphan reference.

### 44. What would you do if an order arrives before its customer record?

Investigate whether the customer record is delayed or missing. Depending on the agreed pipeline design, hold or quarantine the order, retry after the parent arrives, and alert the source owner. Do not invent a customer.

---

## 6. Scenario-based interview questions

### 45. Scenario: Two customers have the same name. How do you identify them?

**Expected answer:**

Use a unique customer identifier such as customer_id. A person's name is not guaranteed to be unique.

**Reasoning:** Names are descriptive attributes, not reliable unique identifiers.

### 46. Scenario: A customer places 20 orders. How would you store them?

**Expected answer:**

Store the customer once in Customers and store 20 records in Orders, each referencing the same customer_id.

**Reasoning:** This is a one-to-many relationship.

### 47. Scenario: An order contains five products. Would you create five order records?

**Expected answer:**

No. Create one Orders record and five Order_Items records, one for each product line.

**Reasoning:** Orders stores order-level information. Order_Items stores product-level details within an order.

### 48. Scenario: The current price of a product changes from 1000 to 1200. What happens to an old order?

**Expected answer:**

The historical order price should remain 1000 if that was the price charged. Store the order-time unit price in Order_Items separately from Products.current_price.

### 49. Scenario: A source order refers to customer_id 999, but that customer does not exist in the target database. What do you do?

**Expected answer:**

Check whether the customer record is delayed, missing or invalid. Quarantine or reject the order according to the pipeline's data-quality policy, report the issue and retry when appropriate.

**Do not:** Silently create a fake customer or drop the record without recording the reason.

### 50. Scenario: A business wants to store multiple phone numbers for each customer. What design would you consider?

**Expected answer:**

If the number of phone records can vary, create a separate Customer_Phones table with phone_id, customer_id and phone_number. This gives a one-to-many relationship.

**Reasoning:** Avoid columns such as phone1, phone2 and phone3 when the number of values is not fixed.

### 51. Scenario: A student can join multiple courses, and each course has many students. How do you design it?

**Expected answer:**

Create Students, Courses and Student_Courses. Student_Courses contains student_id and course_id as foreign keys, plus enrollment attributes such as enrollment_date.

### 52. Scenario: A database contains duplicate customer records. What do you check first?

**Expected answer:**

Check the business definition of a duplicate, candidate identifiers, source-system behavior and whether the records are genuinely different people. Do not delete rows based only on matching names.

---

## 7. Troubleshooting and follow-up questions

### 53. A foreign-key insert fails. What could be the reason?

Possible reasons:

- The referenced parent row does not exist.
- The foreign-key value has a data-type or value mismatch.
- The parent key definition or relationship is incorrect.
- The insert order is wrong and the parent has not been loaded yet.

### 54. A primary-key insert fails. What could be the reason?

The key value may already exist, or the supplied value may be NULL when the primary key requires a value.

### 55. A report shows duplicate customer information after a join. Is the database necessarily wrong?

Not necessarily. A one-to-many join naturally repeats parent attributes for each matching child row. Check the relationship cardinality and aggregation logic before treating it as duplicate source data.

### 56. Why can a customer table have fewer rows than an order table?

One customer can place multiple orders. The number of orders can therefore be greater than the number of customers.

### 57. What is an orphan record?

A child record whose foreign-key value has no matching parent record.

### 58. What is the difference between schema and data?

Schema is the structure and rules. Data is the actual records stored under that structure.

### 59. Why should database design be reviewed before creating tables?

Reviewing the entities, keys, relationships and business rules early prevents structural mistakes that become expensive to fix after applications and pipelines depend on them.

### 60. Does every database need the same data types and SQL syntax?

No. SQL is widely shared, but data types, identity generation, procedural features and some syntax differ between DBMS products. Always check the target database.

---

## 8. Interviewer follow-up drill

Be prepared to explain these follow-ups:

1. If email is unique today, can it be a primary key? What happens if a customer changes email?
2. Why might a generated customer_id be safer than an email address?
3. Can one foreign-key column refer to multiple parent tables?
4. Why is a many-to-many relationship usually resolved with a junction table?
5. What attributes belong in Order_Items rather than Products?
6. What is the difference between a missing parent record and a delayed parent record?
7. Why is a postal code not a good numeric measure?
8. How would you validate relationships before loading a large batch?
9. What would happen if a parent customer is deleted while orders still refer to it?
10. How would you explain a one-to-many relationship to a non-technical stakeholder?

---

## 9. Strong answers: explain the reasoning

Use this structure when answering design questions:

1. **State the business rule.** Example: one customer can place many orders.
2. **Identify the entities.** Customers and Orders.
3. **Choose identifiers.** customer_id and order_id.
4. **Place the foreign key.** Orders.customer_id.
5. **Explain integrity.** The order must refer to a valid customer.
6. **Test realistic cases.** One customer with zero, one and multiple orders.

This structure is stronger than memorizing definitions because it shows that you can apply database concepts.

---

## 10. Quick revision sheet

| Question | Short answer |
|---|---|
| What is DBMS? | Software that manages databases |
| What is a relational database? | Data organized into related tables |
| What is a row? | One record |
| What is a column? | One attribute |
| What is a schema? | Database structure |
| What is a primary key? | Unique row identifier |
| What is a foreign key? | Reference to another table's key |
| Can a foreign key repeat? | Yes |
| Can a primary key be NULL? | No |
| What is a composite key? | Key containing multiple columns |
| What is a surrogate key? | Generated technical identifier |
| What is a natural key? | Business-meaningful identifier |
| What is 1:1? | One record to one record |
| What is 1:M? | One record to many records |
| What is M:N? | Many records to many records |
| What is a junction table? | Table connecting many-to-many entities |
| What is NULL? | Missing, unknown or not-applicable value |
| What is an orphan record? | Child reference without a matching parent |
| What is an ERD? | Diagram of entities and relationships |

---

## 11. How to present the Day 19 mini-project in an interview

**Project:** Operational Database Design

**Sample explanation:**

"I designed a relational database for an online retail application. I identified Customers, Products, Orders and Order_Items as the main entities. I used a primary key in each table and foreign keys to represent the relationships. Customers to Orders is one-to-many. Orders and Products form a many-to-many relationship, which I resolved using Order_Items. I stored quantity and the order-time unit price in Order_Items so that historical sales remain accurate even if the product's current price changes. I also identified business rules such as valid customer references and positive quantities."

**Why this answer is good:**

- It explains the business context.
- It names the entities.
- It explains primary and foreign keys.
- It describes relationship cardinality.
- It includes a real design decision.
- It demonstrates awareness of historical data correctness.

---

## 12. Day 19 completion checklist

- [ ] I can explain database and DBMS.
- [ ] I can identify tables, rows and columns.
- [ ] I can explain schema and data types.
- [ ] I can distinguish primary key and foreign key.
- [ ] I can explain candidate, alternate, natural, surrogate and composite keys.
- [ ] I can identify one-to-one, one-to-many and many-to-many relationships.
- [ ] I can explain why a junction table is needed.
- [ ] I can read the TechStore ERD.
- [ ] I can explain NULL and referential integrity.
- [ ] I can present the Operational Database Design mini-project.

**Next lesson:** Day 20 — SQL DDL, DML and Constraints.
