# Day 20 Interview Preparation: SQL DDL, DML and Constraints

**Pathway:** TechPhile.ai — Data Engineer Career Pathway  
**Phase:** 3 — SQL and Database Engineering  
**Level:** Beginner to Junior Data Engineer  
**Syllabus topic:** SQL DDL, DML and Constraints  
**Mini-project:** Database Creation Lab  
**Primary SQL dialect:** PostgreSQL

---

## 1. How to use this interview guide

Do not memorize one-line definitions only. For each question:

1. Explain the concept in your own words.
2. Give a small SQL example.
3. Explain what the database does.
4. Mention one real-world use or risk.

Use the TechStore customers, products, orders and order_items tables as your common example.

---

## 2. Core SQL questions

### 1. What is SQL?

SQL stands for Structured Query Language. It is used to communicate with relational database systems. A Data Engineer uses SQL to create tables, load data, retrieve records, update data and enforce data quality rules.

### 2. What is the difference between a database and SQL?

A database stores organized data. SQL is the language used to interact with a relational database. PostgreSQL and MySQL are database management systems that understand SQL dialects.

### 3. What is DDL?

DDL means Data Definition Language. It defines or changes database objects such as tables. Common commands are CREATE, ALTER and DROP.

### 4. What is DML?

DML means Data Manipulation Language. It works with records stored in tables. Common commands are INSERT, UPDATE and DELETE.

### 5. DDL versus DML?

DDL changes the database structure. DML changes the data stored inside that structure. CREATE TABLE is DDL; INSERT INTO is DML.

### 6. What does CREATE TABLE do?

It creates a table and defines its columns, data types and constraints.

```sql
CREATE TABLE products (
    product_id INTEGER,
    product_name VARCHAR(100)
);
```

### 7. What is a data type?

A data type defines what kind of value a column can store, such as INTEGER, VARCHAR, NUMERIC, DATE or BOOLEAN.

### 8. Why use NUMERIC for price?

NUMERIC stores exact decimal values within its declared precision and scale. Floating-point types can represent decimal fractions approximately, so they are usually not the first choice for exact monetary amounts.

### 9. What does ALTER TABLE do?

It changes an existing table's structure, for example adding, renaming or dropping a column, or changing a column type.

### 10. What does DROP TABLE do?

It removes the table object and its stored data. It is a destructive operation and should be used only when the target and impact are understood.

---

## 3. Constraint questions

### 11. What is a constraint?

A constraint is a rule enforced by the database to protect data correctness and relationships.

### 12. What is NOT NULL?

It prevents a column from storing NULL. Use it when a value is mandatory.

### 13. What is UNIQUE?

It prevents duplicate values under the defined uniqueness rule. In PostgreSQL, a normal UNIQUE constraint allows multiple NULL values. Combine UNIQUE and NOT NULL if both are required.

### 14. What is a primary key?

A primary key uniquely identifies every row in a table. It cannot contain duplicate or NULL values.

### 15. Can a table have more than one primary key?

A table has one primary key constraint. That primary key can contain more than one column; this is called a composite primary key.

### 16. What is a foreign key?

A foreign key is a column or set of columns that references a key in another table (or, in some designs, the same table). It protects referential integrity.

### 17. Can a foreign key contain duplicate values?

Yes. Many orders can refer to the same customer. The foreign key is not required to be unique.

### 18. Can a foreign key contain NULL?

Yes, if the column permits NULL and the relationship is optional. If every order must belong to a customer, define customer_id as NOT NULL as well as a foreign key.

### 19. What is CHECK?

CHECK ensures that a value satisfies a Boolean condition.

```sql
price NUMERIC(10, 2) CHECK (price >= 0)
```

This rejects negative prices. A CHECK constraint should represent a clear rule and should not be treated as a substitute for all validation.

### 20. What is DEFAULT?

DEFAULT supplies a value when an INSERT statement does not specify a value for that column. Explicitly inserting NULL is not the same as omitting the column.

### 21. What is referential integrity?

It means relationships between related tables remain valid. For example, an order's customer_id must refer to an existing customer when a foreign key is enforced.

### 22. What happens if a foreign key references a missing parent row?

The database rejects the insert or update that would create the invalid reference, unless the constraint has been configured differently or deferred under supported transaction rules.

### 23. What is the difference between PRIMARY KEY and UNIQUE?

Both enforce uniqueness. A primary key also disallows NULL and identifies the row as the table's primary identifier. A table can have multiple UNIQUE constraints but only one primary key constraint.

### 24. Can a table have multiple foreign keys?

Yes. order_items can have one foreign key to orders and another to products.

---

## 4. DML questions

### 25. What does INSERT do?

It adds new records to a table.

```sql
INSERT INTO products (product_id, product_name)
VALUES (101, 'Keyboard');
```

### 26. Why specify column names in INSERT?

It makes the mapping between supplied values and table columns explicit. It also makes the statement less fragile when the table structure changes.

### 27. How do you insert multiple rows?

```sql
INSERT INTO products (product_id, product_name)
VALUES
    (101, 'Keyboard'),
    (102, 'Mouse');
```

### 28. What does UPDATE do?

It changes values in existing rows.

```sql
UPDATE products
SET product_name = 'Wireless Mouse'
WHERE product_id = 102;
```

### 29. What happens if UPDATE has no WHERE clause?

Every row in the target table is updated. Always preview the intended rows using SELECT and the same WHERE condition before executing a high-impact update.

### 30. What does DELETE do?

It removes rows from a table. The table definition remains.

### 31. What happens if DELETE has no WHERE clause?

All rows are deleted from the table, but the table itself remains.

### 32. DELETE versus DROP TABLE?

DELETE removes rows. DROP TABLE removes the table object and its data.

### 33. DELETE versus TRUNCATE?

DELETE can remove selected rows using WHERE. TRUNCATE removes all rows and keeps the table structure. Locking, identity reset and transaction behavior vary by DBMS.

---

## 5. Practical SQL questions

### 34. Create a table with a primary key and required name.

```sql
CREATE TABLE departments (
    department_id INTEGER PRIMARY KEY,
    department_name VARCHAR(100) NOT NULL
);
```

### 35. Add a column to an existing table.

PostgreSQL example:

```sql
ALTER TABLE departments
ADD COLUMN location VARCHAR(80);
```

### 36. Insert a row.

```sql
INSERT INTO departments (department_id, department_name, location)
VALUES (1, 'Engineering', 'Hyderabad');
```

### 37. Update one department's location.

```sql
UPDATE departments
SET location = 'Bengaluru'
WHERE department_id = 1;
```

### 38. Delete one department.

```sql
DELETE FROM departments
WHERE department_id = 1;
```

### 39. Create a CHECK constraint for positive quantity.

```sql
quantity INTEGER CHECK (quantity > 0)
```

### 40. Create a foreign key.

```sql
FOREIGN KEY (customer_id)
REFERENCES customers(customer_id)
```

---

## 6. Scenario-based questions

### 41. An application inserts a customer with no email, but email is mandatory. What should happen?

The database should reject the row if email is declared NOT NULL. The application should also validate input and show a useful error rather than relying only on a low-level database message.

### 42. Two customers try to register with the same email. How do you prevent duplicates?

Use a UNIQUE constraint on email. If email is mandatory, combine UNIQUE with NOT NULL. Application-side checks can improve the user experience, but the database constraint protects against concurrent duplicate inserts.

### 43. An order refers to customer_id 999, but customer 999 does not exist. What should happen?

A foreign key should reject the order. The pipeline or application should verify that the parent customer record exists or handle the rejected record according to its data-quality policy.

### 44. A product price becomes negative due to a bad source file. How can the database protect itself?

Define CHECK (price >= 0). The database rejects negative values even if an upstream script fails to validate them.

### 45. A developer runs UPDATE products SET price = 0. What is the risk?

Without WHERE, every product price becomes zero. Preview target rows, use a safe transaction where appropriate and verify the affected row count.

### 46. A team wants to remove one invalid customer record. Should they DROP the table?

No. DROP removes the entire table. Use DELETE with a precise WHERE condition, after checking dependencies and following the approved data correction process.

### 47. A new requirement asks for a phone number column. What command is suitable?

ALTER TABLE customers ADD COLUMN phone VARCHAR(...). Consider whether the field is mandatory for existing rows and whether a default or staged migration is required.

### 48. Why should a Data Engineer define constraints in the database if Python already validates data?

Multiple applications and jobs may write to the same database. Database constraints provide a shared enforcement boundary. Python validation gives earlier, clearer feedback, while database constraints protect the stored data.

### 49. A setup SQL script fails on its second execution because a table already exists. What should you do?

Inspect the database state first. Use a deliberate migration strategy or a safe idempotent setup approach where appropriate. Do not blindly drop existing tables because they may contain useful data.

### 50. What should you verify after a data load?

Check command results, row counts, key uniqueness, required values, foreign key relationships, business constraints and sample records. Reconcile expected and actual counts.

---

## 7. PostgreSQL and MySQL awareness

SQL concepts are widely shared, but syntax and behavior are not identical across database engines.

| Topic | PostgreSQL | MySQL |
|---|---|---|
| Variable-length text | VARCHAR(n), TEXT | VARCHAR(n), TEXT |
| Exact decimal | NUMERIC(p,s), DECIMAL(p,s) | DECIMAL(p,s) |
| Boolean | BOOLEAN | BOOLEAN is commonly represented as TINYINT(1) |
| Add column | ALTER TABLE t ADD COLUMN c type | ALTER TABLE t ADD COLUMN c type |
| Change type | ALTER TABLE t ALTER COLUMN c TYPE type | ALTER TABLE t MODIFY COLUMN c type |
| Identity/auto-generated key | IDENTITY syntax or sequence-backed approach | AUTO_INCREMENT |

In interviews, explain the concept first and then mention the database-specific syntax when asked. Do not assume every SQL statement behaves identically in every engine.

---

## 8. Mini-project defense: Database Creation Lab

Be prepared to answer:

1. Why did you create separate customers, products, orders and order_items tables?
2. What is the primary key of each table?
3. Which columns are foreign keys?
4. Why is email unique in your training schema?
5. Why is price constrained to be non-negative?
6. Why does order_items store unit_price?
7. What happens when an invalid customer_id is inserted into orders?
8. How did you verify that the four tables contain the expected number of records?
9. Which command would you use to add a new column?
10. How would you safely correct one wrong record?

**Strong project explanation example:**

"I created four related tables for customers, products, orders and order_items. Each table has a primary key. Orders references customers, and order_items references both orders and products. I used NOT NULL for mandatory fields, UNIQUE for customer email, and CHECK constraints for price and quantity. I inserted parent records before child records and verified the row counts. I also tested invalid records to confirm that the database rejected them."

---

## 9. Common interview mistakes

- Saying SQL is a database. SQL is a language.
- Saying DDL and DML are the same.
- Saying a foreign key must be unique.
- Saying UNIQUE always means NOT NULL.
- Forgetting that a primary key is both unique and non-null.
- Claiming DELETE removes the table.
- Forgetting the risk of UPDATE or DELETE without WHERE.
- Using FLOAT for exact monetary amounts without explaining precision.
- Believing Python validation makes database constraints unnecessary.
- Giving syntax without explaining the result.

---

## 10. Rapid revision

| Question | Short answer |
|---|---|
| Create table? | CREATE TABLE |
| Change table structure? | ALTER TABLE |
| Remove table? | DROP TABLE |
| Add records? | INSERT |
| Change records? | UPDATE |
| Remove records? | DELETE |
| Require a value? | NOT NULL |
| Prevent duplicate values? | UNIQUE |
| Identify each row? | PRIMARY KEY |
| Protect table relationships? | FOREIGN KEY |
| Enforce a condition? | CHECK |
| Supply omitted value? | DEFAULT |

---

## 11. Final readiness checklist

- [ ] I can explain SQL, DDL and DML.
- [ ] I can create a table with appropriate data types.
- [ ] I can explain NOT NULL, UNIQUE, PRIMARY KEY, FOREIGN KEY and CHECK.
- [ ] I can insert one row and multiple rows.
- [ ] I can update a selected row safely.
- [ ] I can delete selected rows safely.
- [ ] I can alter a table.
- [ ] I can explain DELETE versus TRUNCATE versus DROP.
- [ ] I can explain why constraints protect data quality.
- [ ] I can defend the Database Creation Lab design.

**Next lesson:** Day 21 — SQL Data Extraction: SELECT, WHERE, ORDER BY, LIMIT, DISTINCT, INNER JOIN and LEFT JOIN.
