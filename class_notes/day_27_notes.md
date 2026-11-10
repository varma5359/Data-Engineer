# Day 27: Database Objects and Automation
**Phase:** 3 — SQL and Database Engineering  
**Mini-project:** Database Automation Lab  
**Learning time:** 2–3 hours  
**Audience:** Beginners progressing to junior Data Engineer level  
**SQL dialect:** PostgreSQL for the main practical examples. MySQL syntax and behavior may differ.

---

## 1. Why do databases have objects beyond tables?

A database is not only a collection of tables. Database systems provide objects that help organize, reuse and automate work.

Imagine TechStore's support team repeatedly needs a report containing order ID, customer name, order date and status. If every analyst writes a different query, logic can drift and access may be inconsistent.

A database view can give users a reusable named query.

A database may also use:
- Functions to calculate or return values.
- Procedures to execute a sequence of database operations.
- Triggers to respond automatically to table events.
- Sequences or identity columns to generate identifiers.
- Indexes to support efficient access.

Today's lesson focuses on views, functions, procedures and triggers.

### Learning outcomes

By the end, you can:
1. Explain common database objects.
2. Create and query a view.
3. Understand what a normal view stores.
4. Explain a materialized view at a high level.
5. Create a simple PostgreSQL function.
6. Understand function parameters and return values.
7. Create and call a stored procedure.
8. Understand the difference between a function and a procedure.
9. Explain what a trigger is.
10. Create a simple audit trigger.
11. Explain BEFORE and AFTER trigger timing.
12. Identify safe and unsafe automation use cases.
13. Inspect and remove database objects.

---

## 2. Database object overview

| Object | Simple meaning | Example use |
|---|---|---|
| Table | Stores rows and columns | customers |
| View | Named query that can be selected from | order summary |
| Materialized view | Stores query results physically | daily report snapshot |
| Function | Reusable logic that returns a value or table | calculate line total |
| Procedure | Callable routine that performs an operation | close a batch |
| Trigger | Automatically runs in response to a table event | write audit record |
| Index | Data structure for locating rows | customer lookup |
| Sequence | Generates numeric values | identifier generation |

Objects are created inside a database schema. A schema is a namespace that helps organize objects.

---

## 3. Prepare a separate lab schema

We will not modify the original TechStore training tables. Create a separate schema for this day's examples.

```sql
CREATE SCHEMA IF NOT EXISTS automation_lab;
```

Create two tables:

```sql
CREATE TABLE automation_lab.customers (
    customer_id INTEGER PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    city VARCHAR(80)
);

CREATE TABLE automation_lab.orders (
    order_id INTEGER PRIMARY KEY,
    customer_id INTEGER NOT NULL
        REFERENCES automation_lab.customers(customer_id),
    order_date DATE NOT NULL,
    order_status VARCHAR(30) NOT NULL,
    order_amount NUMERIC(12, 2) NOT NULL CHECK (order_amount >= 0)
);
```

Insert sample records:

```sql
INSERT INTO automation_lab.customers
    (customer_id, customer_name, city)
VALUES
    (1, 'Asha', 'Hyderabad'),
    (2, 'Ravi', 'Chennai'),
    (3, 'Meena', 'Bengaluru');

INSERT INTO automation_lab.orders
    (order_id, customer_id, order_date, order_status, order_amount)
VALUES
    (5001, 1, '2026-11-01', 'PLACED', 2600),
    (5002, 2, '2026-11-02', 'SHIPPED', 8500),
    (5003, 1, '2026-11-03', 'PLACED', 3000);
```

Verify:

```sql
SELECT COUNT(*) FROM automation_lab.customers;
SELECT COUNT(*) FROM automation_lab.orders;
```

Expected counts: 3 customers and 3 orders.

---

## 4. Views: save a query with a name

A normal view is a named SQL query. When you select from the view, the database runs the view definition as part of the query.

A normal view generally does not store a separate copy of its result rows.

### 4.1 First create a view

Business request: Support staff need order details with customer names.

```sql
CREATE VIEW automation_lab.vw_order_details AS
SELECT
    o.order_id,
    c.customer_name,
    c.city,
    o.order_date,
    o.order_status,
    o.order_amount
FROM automation_lab.orders AS o
JOIN automation_lab.customers AS c
  ON o.customer_id = c.customer_id;
```

Expected: CREATE VIEW.

### 4.2 Read the view

```sql
SELECT *
FROM automation_lab.vw_order_details
ORDER BY order_id;
```

Expected:

| order_id | customer_name | city | order_date | order_status | order_amount |
|---:|---|---|---|---|---:|
| 5001 | Asha | Hyderabad | 2026-11-01 | PLACED | 2600 |
| 5002 | Ravi | Chennai | 2026-11-02 | SHIPPED | 8500 |
| 5003 | Asha | Hyderabad | 2026-11-03 | PLACED | 3000 |

The view gives users a reusable name for a query.

### 4.3 Filter a view

```sql
SELECT order_id, customer_name, order_amount
FROM automation_lab.vw_order_details
WHERE order_status = 'PLACED';
```

Expected: orders 5001 and 5003.

A view can be queried like a table for many common SELECT operations.

### 4.4 Why use views?

- Reuse a common query.
- Hide unnecessary columns from a reporting interface.
- Provide a stable logical interface over underlying tables.
- Simplify repeated joins.
- Help control what information a user can access, when combined with correct permissions.

A view is not automatically a security boundary in every database configuration. Review ownership, privileges and view security behavior.

### 4.5 Does a normal view store data?

A normal PostgreSQL view stores the query definition. It does not materialize a separate result copy. If underlying table data changes, a later query against the view reflects the current underlying data according to transaction visibility.

---

## 5. Materialized views: store a query result

A materialized view stores the result of its defining query physically. It can make repeated analytical reads faster, but the stored result becomes stale until refreshed.

Create one:

```sql
CREATE MATERIALIZED VIEW automation_lab.mv_order_summary AS
SELECT
    customer_id,
    COUNT(*) AS order_count,
    SUM(order_amount) AS total_order_amount
FROM automation_lab.orders
GROUP BY customer_id;
```

Read it:

```sql
SELECT *
FROM automation_lab.mv_order_summary
ORDER BY customer_id;
```

Expected:
- Customer 1: 2 orders, total 5,600.
- Customer 2: 1 order, total 8,500.

Now insert another order:

```sql
INSERT INTO automation_lab.orders
    (order_id, customer_id, order_date, order_status, order_amount)
VALUES
    (5004, 1, '2026-11-04', 'PLACED', 1000);
```

Query the materialized view again. It still contains the old summary until refreshed.

Refresh:

```sql
REFRESH MATERIALIZED VIEW automation_lab.mv_order_summary;
```

Now customer 1 should have 3 orders and total 6,600.

**Key distinction:**
- Normal view: query definition; reads current underlying data.
- Materialized view: stored query result; requires refresh to reflect later changes.

---

## 6. Functions: reusable database logic

A function is a named routine that can accept parameters and return a value or a table. PostgreSQL functions are commonly written in SQL or PL/pgSQL.

### 6.1 First understand a function's purpose

Suppose many queries need to calculate quantity multiplied by unit price. A function can give that business calculation a reusable name.

### 6.2 Create a simple SQL function

```sql
CREATE OR REPLACE FUNCTION automation_lab.calculate_line_total(
    p_quantity INTEGER,
    p_unit_price NUMERIC
)
RETURNS NUMERIC
LANGUAGE SQL
AS $$
    SELECT p_quantity * p_unit_price;
$$;
```

Expected: CREATE FUNCTION.

Understand each part:
- CREATE OR REPLACE FUNCTION creates or replaces a function definition.
- automation_lab.calculate_line_total is its schema-qualified name.
- p_quantity and p_unit_price are input parameters.
- RETURNS NUMERIC declares the returned data type.
- LANGUAGE SQL says the function body is SQL.
- $$ marks the body text.

### 6.3 Call the function

```sql
SELECT automation_lab.calculate_line_total(2, 700);
```

Expected: 1400.

The database passes 2 and 700 into the parameters, multiplies them, and returns 1400.

### 6.4 Use the function with a query

```sql
SELECT
    order_id,
    automation_lab.calculate_line_total(2, 700) AS example_total
FROM automation_lab.orders
ORDER BY order_id;
```

This repeats the same demonstration calculation for each row, so it is not a meaningful production report. It simply shows that a function can be called in a SELECT expression.

A real order-line calculation would pass the actual row's quantity and unit_price.

### 6.5 Function with a conditional rule

A function can contain procedural logic using PL/pgSQL.

```sql
CREATE OR REPLACE FUNCTION automation_lab.order_size_label(
    p_amount NUMERIC
)
RETURNS TEXT
LANGUAGE plpgsql
AS $$
BEGIN
    IF p_amount >= 5000 THEN
        RETURN 'HIGH VALUE';
    ELSE
        RETURN 'STANDARD';
    END IF;
END;
$$;
```

Call it:

```sql
SELECT
    order_id,
    order_amount,
    automation_lab.order_size_label(order_amount) AS order_label
FROM automation_lab.orders
ORDER BY order_id;
```

Expected labels:
- 5001: STANDARD.
- 5002: HIGH VALUE.
- 5003: STANDARD.
- 5004: STANDARD, if the previous materialized-view example inserted it.

This function illustrates parameters, IF, RETURN and END.

---

## 7. Procedures: execute a database operation

A PostgreSQL procedure is called using CALL. Procedures are often used for operations rather than returning a value as a normal function does.

### 7.1 Create a simple procedure

```sql
CREATE OR REPLACE PROCEDURE automation_lab.mark_order_reviewed(
    p_order_id INTEGER
)
LANGUAGE SQL
AS $$
    UPDATE automation_lab.orders
    SET order_status = 'REVIEWED'
    WHERE order_id = p_order_id;
$$;
```

Expected: CREATE PROCEDURE.

### 7.2 Call the procedure

```sql
CALL automation_lab.mark_order_reviewed(5001);
```

Verify:

```sql
SELECT order_id, order_status
FROM automation_lab.orders
WHERE order_id = 5001;
```

Expected: order 5001 has status REVIEWED.

The procedure encapsulates an UPDATE behind a named operation.

### 7.3 Important procedure transaction note

PostgreSQL procedures can have transaction-control capabilities in specific CALL contexts, but transaction control is restricted by how the procedure is invoked and by other conditions. For this beginner example, the procedure simply performs an UPDATE inside the caller's transaction context. Do not assume every procedure can COMMIT internally.

---

## 8. Function versus procedure

| Function | Procedure |
|---|---|
| Invoked in expressions with SELECT in common use | Invoked with CALL |
| Must declare a return type | Does not use a normal function RETURNS value contract |
| Commonly calculates or returns data | Commonly performs an operation |
| Can be used in SELECT expressions when compatible | Not used as a scalar expression |
| Transaction control restrictions differ | PostgreSQL procedures can support transaction control in permitted CALL contexts |

Exact features differ across PostgreSQL, MySQL and other database products. Learn the concept and verify syntax for the database you use.

---

## 9. Triggers: automatically respond to table events

A trigger is database logic that automatically runs when a specified event happens on a table or view.

Common trigger events:
- INSERT
- UPDATE
- DELETE
- TRUNCATE (supported in PostgreSQL trigger definitions)

Common timing:
- BEFORE: runs before the row operation.
- AFTER: runs after the row operation.
- INSTEAD OF: used for certain views to replace the requested operation.

### 9.1 Why might a company use a trigger?

Example: When an order's status changes, the company may need an audit record that records:
- Which order changed.
- Old status.
- New status.
- When it changed.

This can support traceability. Audit design must also consider who can alter or delete audit records and how to retain them securely.

---

## 10. Create an audit table

```sql
CREATE TABLE automation_lab.order_status_audit (
    audit_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    order_id INTEGER NOT NULL,
    old_status VARCHAR(30),
    new_status VARCHAR(30),
    changed_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

This table stores one audit record for each status update that reaches the trigger.

---

## 11. Create a trigger function

In PostgreSQL, a trigger calls a trigger function. A trigger function has no normal declared argument list and returns TRIGGER.

```sql
CREATE OR REPLACE FUNCTION automation_lab.log_order_status_change()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    IF OLD.order_status IS DISTINCT FROM NEW.order_status THEN
        INSERT INTO automation_lab.order_status_audit (
            order_id,
            old_status,
            new_status
        )
        VALUES (
            OLD.order_id,
            OLD.order_status,
            NEW.order_status
        );
    END IF;

    RETURN NEW;
END;
$$;
```

Understand:
- OLD represents the row before the update.
- NEW represents the row after the update.
- IS DISTINCT FROM compares values safely even when NULL is involved.
- INSERT writes an audit row.
- RETURN NEW tells the row-level BEFORE/AFTER trigger what row value to return; for this AFTER trigger, the returned value is ignored, but trigger functions still require a return value.

---

## 12. Create the trigger

```sql
CREATE TRIGGER trg_log_order_status_change
AFTER UPDATE OF order_status
ON automation_lab.orders
FOR EACH ROW
EXECUTE FUNCTION automation_lab.log_order_status_change();
```

Read the definition:
- CREATE TRIGGER defines the trigger.
- AFTER means after the update operation.
- UPDATE OF order_status means it fires for updates that target this column.
- ON names the table.
- FOR EACH ROW means once for each affected row.
- EXECUTE FUNCTION identifies the trigger function.

---

## 13. Test the trigger

First inspect the current status:

```sql
SELECT order_id, order_status
FROM automation_lab.orders
WHERE order_id = 5002;
```

Update the status:

```sql
UPDATE automation_lab.orders
SET order_status = 'DELIVERED'
WHERE order_id = 5002;
```

Inspect the audit row:

```sql
SELECT
    order_id,
    old_status,
    new_status,
    changed_at
FROM automation_lab.order_status_audit
WHERE order_id = 5002
ORDER BY audit_id;
```

Expected: one audit row with old_status SHIPPED and new_status DELIVERED, assuming that was the order's status before the update.

The timestamp depends on the time the update ran.

### 13.1 Update to the same status

```sql
UPDATE automation_lab.orders
SET order_status = 'DELIVERED'
WHERE order_id = 5002;
```

The trigger function checks IS DISTINCT FROM. Since old and new status are the same, it does not insert a new audit row.

This avoids audit noise for no-op changes.

---

## 14. BEFORE versus AFTER trigger

A BEFORE trigger can validate or modify NEW before the row is stored.

An AFTER trigger is suitable for actions that should happen after the row operation, such as writing an audit row in the same transaction.

Important: if the transaction rolls back, the audit insert performed by the trigger also rolls back because it is part of the same transaction. A database trigger is not an independent external event system.

---

## 15. When should triggers be used carefully?

Triggers can be powerful, but hidden automatic behavior can surprise developers.

Risks:
- An INSERT may cause extra updates or audit writes.
- A trigger may create recursion if it updates the same table.
- Performance may be harder to understand.
- Bulk loads may fire triggers for many rows.
- Business logic can become spread across application code and database objects.
- Permissions and ownership must be controlled.

Prefer explicit application or pipeline logic when it makes the process easier to understand and operate. Use triggers when the database-level guarantee is important and the team can maintain the trigger safely.

---

## 16. Inspect and remove database objects

Inspect views:

```sql
SELECT table_schema, table_name
FROM information_schema.views
WHERE table_schema = 'automation_lab';
```

Inspect routines:

```sql
SELECT routine_schema, routine_name, routine_type
FROM information_schema.routines
WHERE routine_schema = 'automation_lab';
```

Drop a view:

```sql
DROP VIEW IF EXISTS automation_lab.vw_order_details;
```

Drop a materialized view:

```sql
DROP MATERIALIZED VIEW IF EXISTS automation_lab.mv_order_summary;
```

Drop a trigger:

```sql
DROP TRIGGER IF EXISTS trg_log_order_status_change
ON automation_lab.orders;
```

Drop a procedure:

```sql
DROP PROCEDURE IF EXISTS automation_lab.mark_order_reviewed(INTEGER);
```

Drop a function:

```sql
DROP FUNCTION IF EXISTS automation_lab.calculate_line_total(INTEGER, NUMERIC);
```

Dropping an object can break dependent queries or applications. Inspect dependencies and permissions before removing objects in a shared database.

---

## 17. Mini-project: Database Automation Lab

### Business requirement

Create a small reusable database layer for TechStore support reporting and order-status auditing.

### Task 1: Create a reusable order details view

Use customers and orders to create vw_order_details. Verify that the view returns three original orders (or four if you inserted order 5004).

### Task 2: Create a materialized customer summary

Create a materialized view with order count and total order amount by customer. Insert a new order and prove the materialized view remains unchanged until REFRESH MATERIALIZED VIEW.

### Task 3: Create a calculation function

Create calculate_line_total(quantity, unit_price) and call it with 2 and 700. Expected result: 1400.

### Task 4: Create a label function

Create order_size_label(amount). Test values below 5000 and at or above 5000.

### Task 5: Create a procedure

Create mark_order_reviewed(order_id). Call it for one order and verify the status changed.

### Task 6: Create an audit table and trigger

Create order_status_audit, the trigger function and the AFTER UPDATE trigger. Change an order status and verify old and new values are captured.

### Task 7: Test a no-op update

Set the status to its existing value. Verify that the trigger function does not create an additional audit row.

### Task 8: Explain transaction behavior

Update an order status inside a transaction and then ROLLBACK. Verify both the order status and the trigger's audit row are rolled back.

### Project checklist

- [ ] I can create and query a view.
- [ ] I can explain normal view versus materialized view.
- [ ] I refreshed the materialized view after source data changed.
- [ ] I created and called a SQL function.
- [ ] I created and called a procedure.
- [ ] I created a trigger function.
- [ ] I created and tested an AFTER UPDATE trigger.
- [ ] I understand OLD and NEW.
- [ ] I tested a no-op update.
- [ ] I verified that trigger changes follow the transaction rollback.

---

## 18. Common beginner mistakes

1. Thinking a normal view stores a separate copy of rows.
2. Thinking a materialized view automatically refreshes.
3. Confusing a function with a procedure.
4. Calling a PostgreSQL procedure as if it were a scalar function.
5. Forgetting a PostgreSQL trigger needs a trigger function.
6. Confusing OLD and NEW.
7. Assuming an AFTER trigger runs outside the transaction.
8. Creating a trigger without checking whether it will recurse.
9. Forgetting that a trigger may run for every affected row.
10. Dropping an object without checking dependencies.
11. Assuming PostgreSQL and MySQL routine syntax is identical.
12. Hiding too much business logic in triggers.

---

## 19. Practice questions

1. What is a database view?
2. Does a normal PostgreSQL view store rows?
3. What is a materialized view?
4. Why must a materialized view be refreshed?
5. What is a database function?
6. What is a stored procedure?
7. How do PostgreSQL functions and procedures differ?
8. What is a trigger?
9. What is the difference between BEFORE and AFTER triggers?
10. What do OLD and NEW represent?
11. What does FOR EACH ROW mean?
12. Why can a trigger create hidden side effects?
13. Does a trigger audit row survive if the surrounding transaction rolls back?
14. Why should triggers be tested during bulk loads?
15. How do you inspect and safely remove database objects?

### Answer hints

1. Named query definition.
2. No, a normal view stores the query definition.
3. Stored query result.
4. It does not automatically reflect later source changes.
5. Reusable routine that returns a value or table.
6. Callable routine for performing operations.
7. PostgreSQL functions are used in expressions and return values; procedures are invoked with CALL and have different transaction-control capabilities.
8. Logic automatically invoked by a table/view event.
9. BEFORE runs before the row operation; AFTER runs after it.
10. OLD is the previous row; NEW is the new row.
11. Trigger runs once per affected row.
12. A normal table operation may automatically execute extra SQL.
13. No, the audit insert is part of the same transaction.
14. A trigger may execute for every affected row and affect load performance or behavior.
15. Inspect catalog/information_schema and dependencies before DROP.

---

## 20. Day 27 revision

- A view is a named query.
- A materialized view stores a query result and needs refresh.
- A function is reusable database logic that returns a value or table.
- A procedure is invoked as an operation using CALL in PostgreSQL.
- A trigger runs automatically for a defined event.
- PostgreSQL trigger functions return TRIGGER.
- OLD and NEW represent row versions.
- Trigger writes participate in the surrounding transaction.
- Triggers can simplify auditing but introduce hidden behavior.
- Inspect dependencies before dropping database objects.

**You are ready for Day 28 when you can create and explain a view, function, procedure and audit trigger in the lab schema.**
