# Day 20: SQL DDL, DML and Constraints

**Pathway:** TechPhile.ai — Data Engineer Career Pathway  
**Phase:** 3 — SQL and Database Engineering  
**Level:** Beginner  
**Estimated learning time:** 2–3 hours  
**Syllabus topic:** SQL DDL, DML and Constraints  
**Mini-project:** Database Creation Lab  
**Primary SQL dialect for class:** PostgreSQL (most examples are standard SQL; dialect notes are shown where needed)

---

## 1. Before we start: what will you learn today?

Yesterday, in Day 19, we designed a small operational database for an online store. We identified tables such as Customers, Products, Orders and Order_Items.

Today, we will turn that design into real database objects and store sample records inside them.

By the end of this lesson, you should be able to:

1. Explain what SQL is and why a Data Engineer uses it.
2. Distinguish DDL from DML.
3. Create a database table with CREATE TABLE.
4. Understand columns, data types and constraints in a table definition.
5. Add, change and remove columns using ALTER TABLE.
6. Insert one row and multiple rows using INSERT.
7. Read the effect of UPDATE and DELETE before running them.
8. Explain NOT NULL, UNIQUE, PRIMARY KEY, FOREIGN KEY and CHECK.
9. Inspect table structure and verify inserted data.
10. Complete the Database Creation Lab.

**Important learning rule:** Do not run every command at once. Read one command, run it, inspect the result, and then move to the next command.

---

## 2. First understand SQL

### 2.1 What is SQL?

SQL stands for Structured Query Language.

It is a language used to communicate with relational database systems. We use SQL to define database structures, insert records, read records, update records and delete records.

Think about an online shopping application:

- The customer registers.
- The application stores customer details.
- The customer places an order.
- The application stores the order.
- The customer changes an address.
- The application updates the address.
- An old test record may need to be removed.

SQL provides commands for these database operations.

### 2.2 SQL is not the database

| Term | Meaning | Example |
|---|---|---|
| Database | Organized collection of data | TechStore |
| DBMS | Software that manages databases | PostgreSQL, MySQL |
| SQL | Language used to communicate with relational databases | SELECT, INSERT, CREATE |
| Table | Data organized into rows and columns | customers |
| Row | One record | One customer's details |
| Column | One attribute | customer_name |

### 2.3 Where does a Data Engineer use SQL?

A Data Engineer uses SQL to:

- Create source and staging tables.
- Load records into databases.
- Read data from multiple tables.
- Correct or transform data.
- Enforce data quality rules.
- Investigate failed records.
- Improve query performance.
- Support analytics and warehouse pipelines.

SQL is not only for analysts. It is one of the most important daily tools for a Data Engineer.

---

## 3. Prepare the SQL environment

### 3.1 Which database should we use?

For this lesson, the primary examples use PostgreSQL.

You can run them in:

1. pgAdmin → select your database → Query Tool.
2. PostgreSQL command-line client (psql).
3. Another PostgreSQL-compatible SQL editor connected to your database.

MySQL Workbench is also useful, but a few syntax details differ. Those differences are called out in the lesson.

### 3.2 Create or select a practice database

If you have permission to create databases, run this once while connected to the PostgreSQL server:

```sql
CREATE DATABASE techstore_training;
```

Expected result in pgAdmin: a successful command message such as **CREATE DATABASE**. A database creation command does not return a table of rows.

Now connect your Query Tool to the new `techstore_training` database. Creating a database and connecting to it are two separate steps.

If your trainer has already created the database, do not create it again. Just connect to it.

### 3.3 Understand how to execute SQL

- Select the statement you want to run, or place the cursor inside it.
- Click Execute in pgAdmin, or use the editor's run shortcut.
- Read the Messages / Data Output area.
- If the command fails, read the error message before changing the query.
- Run SELECT queries to verify the actual table contents.

**Do not assume a command succeeded just because you clicked Execute.** Check the result message.

---

## 4. Understand SQL command categories

For this lesson, focus on two important categories.

### 4.1 DDL — Data Definition Language

DDL commands define or change the structure of database objects.

Examples:

- CREATE — create a database object.
- ALTER — change an existing object's structure.
- DROP — remove a database object.
- TRUNCATE — remove all rows from a table while keeping its structure (behavior and transaction support vary by DBMS).

**Simple analogy:** DDL is like designing or changing the shelves in a warehouse.

### 4.2 DML — Data Manipulation Language

DML commands work with the records stored inside tables.

Examples:

- INSERT — add records.
- UPDATE — change existing records.
- DELETE — remove selected records.

**Simple analogy:** DML is like placing products on the shelves, changing their details or removing them.

### 4.3 DDL versus DML

| Point | DDL | DML |
|---|---|---|
| Main purpose | Define database structure | Work with table records |
| Examples | CREATE, ALTER, DROP | INSERT, UPDATE, DELETE |
| Changes | Tables and other objects | Rows inside tables |
| Example task | Create customers table | Add a customer record |

### 4.4 Quick classification

- CREATE TABLE → DDL.
- ALTER TABLE → DDL.
- DROP TABLE → DDL.
- INSERT INTO → DML.
- UPDATE → DML.
- DELETE FROM → DML.

Do not memorize only the full forms. Understand whether the command changes the table design or the data stored inside it.

---

## 5. Create your first table using CREATE TABLE

### 5.1 Start with a simple table

Imagine that TechStore needs to store customer details.

Before storing customers, we need a table structure.

A table definition answers these questions:

- What is the table called?
- What columns does it contain?
- What type of value can each column store?
- Which rules must every record follow?

Run:

```sql
CREATE TABLE customers (
    customer_id INTEGER,
    customer_name VARCHAR(100),
    email VARCHAR(150),
    city VARCHAR(60)
);
```

Expected result: a successful command message such as **CREATE TABLE**.

This command creates the table structure. It does not insert any customer records.

### 5.2 Understand every part of the query

| SQL part | Meaning |
|---|---|
| CREATE TABLE | Ask the database to create a table |
| customers | Name of the new table |
| customer_id | First column |
| INTEGER | Whole-number data type |
| customer_name | Second column |
| VARCHAR(100) | Text with a maximum length of 100 characters |
| email | Email column |
| VARCHAR(150) | Text with a maximum length of 150 characters |
| city | City column |
| VARCHAR(60) | Text with a maximum length of 60 characters |
| Comma | Separates column definitions |
| Semicolon | Ends the SQL statement |

The parentheses contain the column definitions.

### 5.3 What is a data type?

A data type tells the database what kind of value a column is designed to store.

| Data type | Used for | Example |
|---|---|---|
| INTEGER | Whole numbers | 101 |
| BIGINT | Larger whole numbers | 9000000000 |
| VARCHAR(n) | Variable-length text | 'Hyderabad' |
| TEXT | Text without a small declared length limit | 'Customer feedback' |
| NUMERIC(p,s) | Exact decimal values | 1250.50 |
| DATE | Calendar date | '2026-11-10' |
| TIMESTAMP | Date and time | '2026-11-10 10:30:00' |
| BOOLEAN | True or false | TRUE |

**Important:** For prices and money-like values, prefer an exact decimal type such as NUMERIC rather than FLOAT or REAL. Floating-point types can represent some decimal fractions approximately.

### 5.4 Inspect the table structure

In pgAdmin, expand:

Servers → Databases → techstore_training → Schemas → public → Tables → customers → Columns.

You can also use a database metadata query:

```sql
SELECT
    column_name,
    data_type,
    character_maximum_length,
    is_nullable
FROM information_schema.columns
WHERE table_name = 'customers'
ORDER BY ordinal_position;
```

Expected result contains four rows:

| column_name | data_type | character_maximum_length |
|---|---|---:|
| customer_id | integer | NULL |
| customer_name | character varying | 100 |
| email | character varying | 150 |
| city | character varying | 60 |

The exact display of NULL metadata values depends on the SQL client. The important point is that the table has four columns in the order we defined.

### 5.5 Check whether the table contains records

```sql
SELECT * FROM customers;
```

Expected result: zero rows, because we created the table but have not inserted data.

**Remember:** CREATE TABLE creates the structure. INSERT adds records.

---

## 6. Add rules with constraints

A table without constraints may accept records that do not make business sense.

For example:

- A customer may have no customer ID.
- Two customers may use the same unique email address.
- A product may have a negative price.
- An order may refer to a customer who does not exist.

Constraints are database rules that protect data quality.

### 6.1 NOT NULL

NOT NULL means the column must have a value. The database does not allow NULL in that column.

Create a new practice table:

```sql
CREATE TABLE customer_contacts (
    contact_id INTEGER,
    customer_name VARCHAR(100) NOT NULL,
    email VARCHAR(150)
);
```

Expected result: **CREATE TABLE**.

Try inserting a row without the required name:

```sql
INSERT INTO customer_contacts (contact_id, email)
VALUES (1, 'asha@example.com');
```

Expected result: the database rejects the statement with a NOT NULL constraint error.

Why? The customer_name column was declared NOT NULL, but the INSERT did not supply a value for it.

Now insert a valid row:

```sql
INSERT INTO customer_contacts (contact_id, customer_name, email)
VALUES (1, 'Asha', 'asha@example.com');
```

Expected result: **INSERT 0 1** in PostgreSQL, meaning one row was inserted.

### 6.2 UNIQUE

UNIQUE means that duplicate values are not allowed in the constrained column or combination of columns.

A customer email should normally be unique in this practice example.

```sql
CREATE TABLE unique_customers (
    customer_id INTEGER,
    customer_name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE
);
```

Insert the first customer:

```sql
INSERT INTO unique_customers
VALUES (1, 'Asha', 'asha@example.com');
```

Expected result: one row inserted.

Now try to insert another customer with the same email:

```sql
INSERT INTO unique_customers
VALUES (2, 'Ravi', 'asha@example.com');
```

Expected result: rejected with a UNIQUE constraint violation.

**Important NULL note:** In PostgreSQL, a normal UNIQUE constraint permits multiple NULL values because NULL values are not treated as equal for this uniqueness check. If the field must always exist and be unique, combine NOT NULL and UNIQUE.

### 6.3 PRIMARY KEY

A primary key uniquely identifies each row in a table.

A primary key combines two rules:

1. Values must be unique.
2. Values cannot be NULL.

Example:

```sql
CREATE TABLE products (
    product_id INTEGER PRIMARY KEY,
    product_name VARCHAR(120) NOT NULL,
    price NUMERIC(10, 2) NOT NULL
);
```

Expected result: **CREATE TABLE**.

Here:

- product_id identifies a product.
- PRIMARY KEY makes product_id unique and not nullable.
- product_name is mandatory.
- price is mandatory and supports two decimal places.

Insert two products:

```sql
INSERT INTO products (product_id, product_name, price)
VALUES
    (101, 'Keyboard', 1200.00),
    (102, 'Mouse', 650.00);
```

Expected result: **INSERT 0 2**.

Verify:

```sql
SELECT * FROM products ORDER BY product_id;
```

Expected result:

| product_id | product_name | price |
|---:|---|---:|
| 101 | Keyboard | 1200.00 |
| 102 | Mouse | 650.00 |

Try inserting another product with product_id 101:

```sql
INSERT INTO products
VALUES (101, 'USB Cable', 250.00);
```

Expected result: rejected because product_id 101 already exists.

### 6.4 FOREIGN KEY

A foreign key connects one table to a key in another table.

Example:

- customers has customer_id as its primary key.
- orders stores customer_id to identify the customer who placed the order.
- The orders.customer_id value must refer to an existing customers.customer_id value.

First create the parent table:

```sql
CREATE TABLE fk_customers (
    customer_id INTEGER PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL
);
```

Then create the child table:

```sql
CREATE TABLE customer_orders (
    order_id INTEGER PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    order_date DATE NOT NULL,
    CONSTRAINT fk_orders_customer
        FOREIGN KEY (customer_id)
        REFERENCES fk_customers(customer_id)
);
```

Expected result: both CREATE TABLE statements succeed.

Insert a customer:

```sql
INSERT INTO fk_customers
VALUES (10, 'Asha');
```

Now insert an order for customer 10:

```sql
INSERT INTO customer_orders
VALUES (5001, 10, '2026-11-10');
```

Expected result: one row inserted.

Now try an order for a customer that does not exist:

```sql
INSERT INTO customer_orders
VALUES (5002, 999, '2026-11-10');
```

Expected result: rejected with a foreign key violation.

Why? Customer 999 does not exist in fk_customers. The database protects referential integrity.

### 6.5 CHECK

CHECK validates whether a value satisfies a condition.

A product price should not be negative.

```sql
CREATE TABLE checked_products (
    product_id INTEGER PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    price NUMERIC(10, 2) CHECK (price >= 0)
);
```

Insert a valid price:

```sql
INSERT INTO checked_products
VALUES (1, 'Monitor', 8500.00);
```

Expected result: one row inserted.

Try a negative price:

```sql
INSERT INTO checked_products
VALUES (2, 'Damaged Sample', -500.00);
```

Expected result: rejected because -500 is less than zero.

A CHECK constraint is useful for rules such as:

- quantity must be greater than zero.
- discount percentage must be between 0 and 100.
- rating must be between 1 and 5.

A CHECK rule should express a clear business condition. It is not a replacement for every type of validation.

### 6.6 DEFAULT

DEFAULT supplies a value when an INSERT statement does not provide one for that column.

```sql
CREATE TABLE support_tickets (
    ticket_id INTEGER PRIMARY KEY,
    issue_title VARCHAR(150) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'OPEN'
);
```

Insert without specifying status:

```sql
INSERT INTO support_tickets (ticket_id, issue_title)
VALUES (1, 'Laptop network issue');
```

Verify:

```sql
SELECT * FROM support_tickets;
```

Expected result:

| ticket_id | issue_title | status |
|---:|---|---|
| 1 | Laptop network issue | OPEN |

DEFAULT is used when the column is omitted. Explicitly inserting NULL is different; a NOT NULL rule would reject that NULL.

### 6.7 Constraint summary

| Constraint | What it protects |
|---|---|
| NOT NULL | Required value |
| UNIQUE | Duplicate values |
| PRIMARY KEY | Unique, non-null row identity |
| FOREIGN KEY | Valid relationship to another table |
| CHECK | Business condition |
| DEFAULT | Supplies a value when omitted |

---

## 7. Insert data using INSERT

### 7.1 Insert one row

Use the products table created earlier:

```sql
INSERT INTO products (product_id, product_name, price)
VALUES (103, 'Webcam', 2200.00);
```

Expected result: **INSERT 0 1**.

The column list tells the database which columns receive the supplied values. The values must match the column order and compatible data types.

### 7.2 Why should we write column names?

This is valid but less clear:

```sql
INSERT INTO products
VALUES (104, 'Headset', 1800.00);
```

This is clearer and safer:

```sql
INSERT INTO products (product_id, product_name, price)
VALUES (104, 'Headset', 1800.00);
```

If a table structure changes, explicitly naming columns helps prevent accidental value-to-column mismatches.

### 7.3 Insert multiple rows

```sql
INSERT INTO products (product_id, product_name, price)
VALUES
    (105, 'Laptop Stand', 1500.00),
    (106, 'USB Hub', 900.00),
    (107, 'Desk Lamp', 750.00);
```

Expected result: **INSERT 0 3**.

Verify the rows:

```sql
SELECT * FROM products ORDER BY product_id;
```

You should now see product IDs 101 through 107, assuming the earlier examples were run once and in order.

### 7.4 Common INSERT errors

| Error | Likely reason | How to correct it |
|---|---|---|
| Duplicate key | Primary key already exists | Use a new ID or a deliberate upsert |
| NOT NULL violation | Required column was omitted or given NULL | Supply a valid value |
| UNIQUE violation | Unique value already exists | Supply a distinct value |
| Foreign key violation | Referenced parent record is missing | Insert/verify parent first |
| Type error | Value cannot be stored as declared | Correct the value or type |
| Column count mismatch | Number of values does not match columns | Check the column list and VALUES |

---

## 8. Change records using UPDATE

### 8.1 What does UPDATE do?

UPDATE changes values in existing rows.

Suppose the price of the Mouse product changes from 650 to 700.

First inspect the row:

```sql
SELECT * FROM products
WHERE product_id = 102;
```

Expected result:

| product_id | product_name | price |
|---:|---|---:|
| 102 | Mouse | 650.00 |

Now update it:

```sql
UPDATE products
SET price = 700.00
WHERE product_id = 102;
```

Expected result: PostgreSQL reports **UPDATE 1**.

Verify the change:

```sql
SELECT * FROM products
WHERE product_id = 102;
```

Expected result: price is now 700.00.

### 8.2 Understand UPDATE syntax

| Part | Meaning |
|---|---|
| UPDATE products | Select the table to change |
| SET price = 700.00 | Set the new value |
| WHERE product_id = 102 | Identify which row should change |
| Semicolon | End the command |

### 8.3 The most dangerous beginner mistake: forgetting WHERE

Consider:

```sql
UPDATE products
SET price = 100.00;
```

This changes the price of **every row** in the products table.

Before running an UPDATE:

1. Write the WHERE condition.
2. Run a SELECT with the same WHERE condition.
3. Confirm that it returns exactly the intended rows.
4. Run UPDATE.
5. Verify the changed rows.

In a production database, use an approved change process and transaction safeguards. Never experiment on production data.

### 8.4 Update more than one column

```sql
UPDATE products
SET product_name = 'Wireless Mouse',
    price = 950.00
WHERE product_id = 102;
```

Expected result: one row updated, if product_id 102 exists.

---

## 9. Remove records using DELETE

### 9.1 What does DELETE do?

DELETE removes rows from a table. It does not remove the table structure.

Inspect a test row:

```sql
SELECT * FROM products
WHERE product_id = 107;
```

Delete it:

```sql
DELETE FROM products
WHERE product_id = 107;
```

Expected result: **DELETE 1**.

Verify:

```sql
SELECT * FROM products
WHERE product_id = 107;
```

Expected result: zero rows.

### 9.2 DELETE versus DROP

| Command | What it removes |
|---|---|
| DELETE FROM products WHERE ... | Selected rows |
| DELETE FROM products | All rows, table remains |
| DROP TABLE products | Table structure and its data |

### 9.3 Why WHERE matters in DELETE

```sql
DELETE FROM products;
```

This removes all rows from products. The table remains, but it becomes empty.

Do not run this command casually. In a real system, confirm the target, take required backups, follow access approvals and use a transaction where appropriate.

---

## 10. Change table structure using ALTER TABLE

A business requirement may change after a table has been created.

Example: TechStore now wants to store each customer's phone number.

### 10.1 Add a column

```sql
ALTER TABLE customers
ADD COLUMN phone VARCHAR(20);
```

Expected result: **ALTER TABLE**.

The existing rows remain. The new phone column will initially contain NULL for existing rows because we did not provide a default.

Inspect the structure again using information_schema.columns or the pgAdmin table tree.

### 10.2 Rename a column

```sql
ALTER TABLE customers
RENAME COLUMN city TO city_name;
```

Expected result: **ALTER TABLE**.

The column is renamed. Queries that still refer to city must be updated.

### 10.3 Change a column's data type

In PostgreSQL:

```sql
ALTER TABLE customers
ALTER COLUMN phone TYPE VARCHAR(30);
```

Expected result: **ALTER TABLE**, if the existing values can be converted.

Changing a data type on a populated table needs care. Check existing values, dependent views, indexes, constraints and application code before making the change.

### 10.4 Drop a column

```sql
ALTER TABLE customers
DROP COLUMN phone;
```

Expected result: **ALTER TABLE**.

The phone column is removed. This is a destructive schema change; its stored values are lost.

### 10.5 ALTER TABLE syntax varies by database

PostgreSQL and MySQL do not use identical syntax for every ALTER operation. For example, changing a column's type uses different syntax in MySQL. Always check the documentation for the database engine used by your project.

---

## 11. Remove a table using DROP TABLE

DROP TABLE removes the table object itself.

For a disposable practice table:

```sql
CREATE TABLE drop_practice (
    sample_id INTEGER
);
```

Then:

```sql
DROP TABLE drop_practice;
```

Expected result: **DROP TABLE**.

After dropping it, queries against drop_practice fail because the table no longer exists.

### 11.1 DROP versus DELETE versus TRUNCATE

| Command | Removes rows? | Removes table definition? | Typical use |
|---|---|---|---|
| DELETE | Yes | No | Remove selected or all rows |
| TRUNCATE | Yes, all rows | No | Quickly empty a table |
| DROP TABLE | Yes | Yes | Remove the table object |

TRUNCATE behavior, identity reset behavior and transaction details differ across database engines. Learn the behavior of your chosen DBMS before using it in a production workflow.

### 11.2 Use DROP carefully

- Confirm the database and schema.
- Confirm the exact table name.
- Check whether other tables reference it.
- Check downstream applications and reports.
- Back up data if required.
- Never run destructive commands against production without authorization.

---

## 12. Build the TechStore practice schema

Now connect today's commands to the Day 19 design. We will create a small, consistent set of operational tables.

**Practice warning:** Run this setup in a fresh training database. If you already created tables with these names, do not blindly run the script again. Inspect them first or use a clean practice database.

### 12.1 Create customers

```sql
CREATE TABLE customers (
    customer_id INTEGER PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL UNIQUE,
    city VARCHAR(60),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### 12.2 Create products

```sql
CREATE TABLE products (
    product_id INTEGER PRIMARY KEY,
    product_name VARCHAR(120) NOT NULL,
    category VARCHAR(60) NOT NULL,
    price NUMERIC(10, 2) NOT NULL CHECK (price >= 0),
    is_active BOOLEAN NOT NULL DEFAULT TRUE
);
```

### 12.3 Create orders

```sql
CREATE TABLE orders (
    order_id INTEGER PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    order_date DATE NOT NULL,
    order_status VARCHAR(20) NOT NULL
        CHECK (order_status IN ('PLACED', 'SHIPPED', 'CANCELLED')),
    CONSTRAINT fk_orders_customers
        FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
);
```

### 12.4 Create order_items

```sql
CREATE TABLE order_items (
    order_item_id INTEGER PRIMARY KEY,
    order_id INTEGER NOT NULL,
    product_id INTEGER NOT NULL,
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    unit_price NUMERIC(10, 2) NOT NULL CHECK (unit_price >= 0),
    CONSTRAINT fk_items_orders
        FOREIGN KEY (order_id)
        REFERENCES orders(order_id),
    CONSTRAINT fk_items_products
        FOREIGN KEY (product_id)
        REFERENCES products(product_id)
);
```

### 12.5 Why do we have four tables?

- customers stores customer details once.
- products stores product details once.
- orders stores each order header.
- order_items stores the products and quantities belonging to an order.

An order can contain multiple products. Therefore, order_items connects orders and products.

The unit_price in order_items records the price used for that particular sale. The current products.price may change later, but the historical order line should retain the price charged at the time of the order.

### 12.6 Insert sample customers

```sql
INSERT INTO customers (customer_id, customer_name, email, city)
VALUES
    (1, 'Asha', 'asha@example.com', 'Hyderabad'),
    (2, 'Ravi', 'ravi@example.com', 'Chennai'),
    (3, 'Meena', 'meena@example.com', 'Bengaluru');
```

Expected result: **INSERT 0 3**.

### 12.7 Insert sample products

```sql
INSERT INTO products (product_id, product_name, category, price)
VALUES
    (101, 'Keyboard', 'Accessories', 1200.00),
    (102, 'Mouse', 'Accessories', 700.00),
    (103, 'Monitor', 'Electronics', 8500.00),
    (104, 'Laptop Stand', 'Accessories', 1500.00);
```

Expected result: **INSERT 0 4**.

### 12.8 Insert sample orders

```sql
INSERT INTO orders (order_id, customer_id, order_date, order_status)
VALUES
    (5001, 1, '2026-11-01', 'PLACED'),
    (5002, 2, '2026-11-02', 'SHIPPED'),
    (5003, 1, '2026-11-03', 'PLACED');
```

Expected result: **INSERT 0 3**.

### 12.9 Insert order items

```sql
INSERT INTO order_items
    (order_item_id, order_id, product_id, quantity, unit_price)
VALUES
    (1, 5001, 101, 1, 1200.00),
    (2, 5001, 102, 2, 700.00),
    (3, 5002, 103, 1, 8500.00),
    (4, 5003, 104, 2, 1500.00);
```

Expected result: **INSERT 0 4**.

### 12.10 Verify all four tables

Run each query:

```sql
SELECT * FROM customers ORDER BY customer_id;
SELECT * FROM products ORDER BY product_id;
SELECT * FROM orders ORDER BY order_id;
SELECT * FROM order_items ORDER BY order_item_id;
```

Expected row counts:

| Table | Expected rows |
|---|---:|
| customers | 3 |
| products | 4 |
| orders | 3 |
| order_items | 4 |

If the counts differ, stop and investigate before continuing. Common causes include a previous partial run, duplicate primary keys or a command that failed.

---

## 13. Understand what constraints protect in our project

| Table | Rule | Why it matters |
|---|---|---|
| customers | customer_id PRIMARY KEY | Each customer has one identity |
| customers | email NOT NULL UNIQUE | Every customer has a supplied, non-duplicated email in this training design |
| products | price CHECK (price >= 0) | A product cannot have a negative price |
| orders | customer_id FOREIGN KEY | An order must belong to an existing customer |
| orders | order_status CHECK | Only agreed order states are accepted |
| order_items | order_id FOREIGN KEY | Every item belongs to an existing order |
| order_items | product_id FOREIGN KEY | Every item refers to an existing product |
| order_items | quantity CHECK (quantity > 0) | An order line must have a positive quantity |

Constraints are enforced by the database. They protect data even if multiple applications or scripts write to the same table.

---

## 14. Practice: predict the result before executing

For each example, say whether the database should accept or reject the command. Then run it in your training database and compare.

### Exercise 1 — Missing customer name

```sql
INSERT INTO customers (customer_id, email, city)
VALUES (4, 'new@example.com', 'Pune');
```

Expected: rejected. customer_name is NOT NULL.

### Exercise 2 — Duplicate email

```sql
INSERT INTO customers (customer_id, customer_name, email)
VALUES (4, 'New Customer', 'asha@example.com');
```

Expected: rejected. The email already belongs to another row.

### Exercise 3 — Negative price

```sql
INSERT INTO products (product_id, product_name, category, price)
VALUES (105, 'Invalid Product', 'Test', -100.00);
```

Expected: rejected. The price CHECK condition fails.

### Exercise 4 — Unknown customer reference

```sql
INSERT INTO orders (order_id, customer_id, order_date, order_status)
VALUES (5004, 999, '2026-11-04', 'PLACED');
```

Expected: rejected. Customer 999 does not exist.

### Exercise 5 — Zero quantity

```sql
INSERT INTO order_items
    (order_item_id, order_id, product_id, quantity, unit_price)
VALUES
    (5, 5001, 101, 0, 1200.00);
```

Expected: rejected. quantity must be greater than zero.

**Learning point:** A good database design does not merely store data. It prevents many invalid states from entering the system.

---

## 15. Common beginner mistakes and how to correct them

### Mistake 1 — Creating a table but expecting records

CREATE TABLE creates columns and rules. Use INSERT to add records.

### Mistake 2 — Forgetting the column list in INSERT

Always prefer an explicit column list. It makes the query easier to read and safer when table definitions evolve.

### Mistake 3 — Updating without WHERE

An UPDATE without WHERE affects every row. Preview the target rows with SELECT first.

### Mistake 4 — Deleting without WHERE

A DELETE without WHERE removes every row. Check the condition and the target table carefully.

### Mistake 5 — Confusing NULL with zero

NULL means missing or unknown. Zero is a real numeric value. An empty string is also different from NULL.

### Mistake 6 — Assuming UNIQUE means NOT NULL

In PostgreSQL, a normal UNIQUE constraint permits multiple NULL values. Combine UNIQUE with NOT NULL if both rules are required.

### Mistake 7 — Thinking a foreign key must be unique

A foreign key can repeat. Many orders can belong to the same customer.

### Mistake 8 — Dropping a table to remove a few records

Use DELETE with a suitable WHERE condition when only selected records should be removed. DROP removes the table itself.

### Mistake 9 — Running a setup script repeatedly

A second run may fail because tables or primary key values already exist. Understand the database state before rerunning setup statements.

### Mistake 10 — Assuming every database uses identical syntax

PostgreSQL, MySQL, SQL Server and Oracle share many SQL concepts but differ in some data types, identity syntax, ALTER statements and administrative commands.

---

## 16. Practical SQL command revision

| Command | Purpose | Example |
|---|---|---|
| CREATE TABLE | Create a table | CREATE TABLE products (...) |
| ALTER TABLE | Change table structure | ALTER TABLE products ADD COLUMN sku VARCHAR(30) |
| DROP TABLE | Remove a table | DROP TABLE practice_table |
| INSERT | Add records | INSERT INTO products (...) VALUES (...) |
| UPDATE | Change records | UPDATE products SET price = 900 WHERE product_id = 102 |
| DELETE | Remove records | DELETE FROM products WHERE product_id = 102 |
| SELECT | Read records | SELECT * FROM products |

SELECT is included here for verification and will be taught in detail in Day 21.

---

## 17. Mini-project: Database Creation Lab

### 17.1 Business problem

TechStore is starting a small online store. The application needs to store customers, products, orders and the products included in each order.

Your responsibility is to create the database tables and protect them with appropriate constraints.

### 17.2 Required tables

| Table | Required columns |
|---|---|
| customers | customer_id, customer_name, email, city |
| products | product_id, product_name, category, price |
| orders | order_id, customer_id, order_date, order_status |
| order_items | order_item_id, order_id, product_id, quantity, unit_price |

### 17.3 Required constraints

- Every table must have a primary key.
- Customer name and email must be mandatory.
- Customer email must be unique.
- Product price cannot be negative.
- Every order must refer to an existing customer.
- Order status must be one of the approved values.
- Every order item must refer to an existing order and product.
- Quantity must be greater than zero.
- Unit price cannot be negative.

### 17.4 Implementation sequence

1. Connect to the practice database.
2. Create customers.
3. Create products.
4. Create orders after customers exist.
5. Create order_items after orders and products exist.
6. Insert parent records first: customers and products.
7. Insert orders.
8. Insert order items.
9. Verify the number of rows in every table.
10. Try at least three invalid records and observe the constraint errors.

### 17.5 Deliverables

Submit:

- SQL script containing all CREATE TABLE statements.
- SQL script containing sample INSERT statements.
- Screenshot or exported result showing the four tables.
- A short explanation of each primary key and foreign key.
- Evidence of at least three rejected invalid records.
- A short note explaining why the database rejected each invalid record.

### 17.6 Project completion checklist

- [ ] I created all four tables.
- [ ] I used appropriate data types.
- [ ] I defined primary keys.
- [ ] I defined foreign keys.
- [ ] I used NOT NULL, UNIQUE and CHECK where needed.
- [ ] I inserted valid sample records.
- [ ] I verified table row counts.
- [ ] I tested invalid values.
- [ ] I can explain each constraint in simple language.

---

## 18. Day 20 knowledge check

Try answering without looking at the earlier sections.

1. What is SQL?
2. What is the difference between DDL and DML?
3. Which command creates a table?
4. What is the purpose of ALTER TABLE?
5. What is the difference between a table structure and table records?
6. What does NOT NULL prevent?
7. What does UNIQUE prevent?
8. What two properties does a primary key enforce?
9. What does a foreign key protect?
10. What does CHECK do?
11. What is the purpose of DEFAULT?
12. What happens if UPDATE is run without WHERE?
13. What happens if DELETE is run without WHERE?
14. What is the difference between DELETE and DROP TABLE?
15. Why should INSERT statements name their columns?
16. Why should parent records be inserted before child records?
17. Why is a negative product price rejected in our design?
18. Why can a foreign key value appear in many rows?
19. Why should destructive SQL be tested carefully?
20. Why do we verify data after INSERT, UPDATE and DELETE?

### Suggested answers

1. SQL is the language used to communicate with relational databases.
2. DDL defines database structures; DML manipulates records.
3. CREATE TABLE.
4. It changes an existing table's structure.
5. Structure defines columns and rules; records are the stored rows.
6. It prevents missing NULL values in that column.
7. It prevents duplicate values under the uniqueness rule.
8. A primary key is unique and not NULL.
9. It protects valid relationships between tables.
10. It checks whether values satisfy a condition.
11. It supplies a value when an INSERT omits the column.
12. Every row in the table is updated.
13. Every row in the table is deleted, but the table remains.
14. DELETE removes rows; DROP TABLE removes the table object and its data.
15. It makes the intended mapping clear and safer if the schema changes.
16. A child foreign key must refer to an existing parent key.
17. The CHECK condition requires price to be zero or greater.
18. Many child records can refer to the same parent record.
19. They can change or remove important data or schema objects.
20. Verification confirms that the database reached the intended state.

---

## 19. Interview preparation focus

Be ready to explain DDL and DML with examples, create a table from a business requirement, choose data types, define primary and foreign keys, explain NOT NULL / UNIQUE / CHECK, and safely use INSERT, UPDATE and DELETE.

Do not only recite syntax. Interviewers may ask what happens when a constraint is violated or what could go wrong if a WHERE clause is missing.

---

## 20. What comes next?

In Day 21 — SQL Data Extraction, we will learn how to retrieve data using SELECT, filter rows with WHERE, sort with ORDER BY, remove duplicates with DISTINCT, limit results, and combine related tables using INNER JOIN and LEFT JOIN.

Today we created and populated the tables. Tomorrow we will learn how to ask useful questions about the data stored in them.
