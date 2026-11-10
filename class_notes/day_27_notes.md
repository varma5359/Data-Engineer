# Day 27: Database Objects and Automation

**Pathway:** TechPhile.ai — Data Engineer Career Pathway  
**Phase:** 3 — SQL and Database Engineering  
**Audience:** Freshers, college students and junior Data Engineer candidates  
**Level:** Beginner to junior | **Class time:** 2–3 hours  
**Approved syllabus:** Views, stored procedures, triggers, functions, database automation concepts  
**Mini-project:** Database Automation Lab  
**Primary database:** PostgreSQL; MySQL differences are identified where relevant.

---

## 1. How to study

Read one concept, predict what the query or program will do, execute the small example, compare the result and explain it aloud. Do not run the entire lesson as one large script.

## 2. Prerequisite and learning outcomes

Use the TechStore training database from Day 20: customers, products, orders and order_items. Confirm you are connected to the correct practice database. If sample data differs, inspect it before comparing outputs. Never experiment on production data.

By the end, you should understand each item in today's syllabus, execute the main examples, explain the output, recognize common mistakes and complete **Database Automation Lab**.

## 3. Database objects

A database contains tables plus views, indexes, functions, procedures, triggers and sequences. These objects package reusable logic and simplify controlled access.

## 4. Views

A normal PostgreSQL view stores a query definition, not a separate copy of result rows.

```sql
CREATE VIEW vw_order_details AS
SELECT o.order_id,o.order_date,c.customer_name,p.product_name,
       oi.quantity,oi.unit_price,oi.quantity*oi.unit_price AS line_total
FROM orders o JOIN customers c ON c.customer_id=o.customer_id
JOIN order_items oi ON oi.order_id=o.order_id
JOIN products p ON p.product_id=oi.product_id;
SELECT * FROM vw_order_details ORDER BY order_id;
```
Expected four order-item rows. A view is a reusable query interface, not automatically a security boundary. Some simple views are updatable; joined/aggregated views may not be.

## 5. Materialized view

A PostgreSQL materialized view stores query results:
```sql
CREATE MATERIALIZED VIEW mv_order_totals AS
SELECT order_id,SUM(quantity*unit_price) AS total
FROM order_items GROUP BY order_id;
SELECT * FROM mv_order_totals ORDER BY order_id;
REFRESH MATERIALIZED VIEW mv_order_totals;
```
It may speed repeated reads but becomes stale until refreshed. Refresh schedule is part of pipeline design.

## 6. Function

```sql
CREATE OR REPLACE FUNCTION calculate_line_total(
 p_quantity INTEGER,p_unit_price NUMERIC
)
RETURNS NUMERIC LANGUAGE SQL
AS $$ SELECT p_quantity*p_unit_price; $$;
SELECT calculate_line_total(2,700.00);
```
Expected 1400. RETURNS declares result type; parameters are inputs; LANGUAGE SQL defines body language; $$ delimits body.

## 7. Procedure

```sql
CREATE OR REPLACE PROCEDURE set_product_price(
 p_product_id INTEGER,p_new_price NUMERIC
)
LANGUAGE SQL AS $$
 UPDATE products SET price=p_new_price
 WHERE product_id=p_product_id;
$$;
CALL set_product_price(102,725.00);
SELECT product_id,price FROM products WHERE product_id=102;
```
Expected price 725. Production routine must validate price, missing product, audit and caller privileges.

Functions return a declared result and are used in expressions; procedures are invoked with CALL and perform operations. Syntax/capabilities vary by DBMS.

## 8. Triggers

A trigger runs logic in response to INSERT/UPDATE/DELETE. Audit price changes:

```sql
CREATE TABLE product_price_audit(
 audit_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
 product_id INTEGER NOT NULL,old_price NUMERIC(10,2),
 new_price NUMERIC(10,2),
 changed_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

Trigger function:
```sql
CREATE OR REPLACE FUNCTION log_product_price_change()
RETURNS TRIGGER LANGUAGE PLPGSQL AS $$
BEGIN
 IF OLD.price IS DISTINCT FROM NEW.price THEN
  INSERT INTO product_price_audit(product_id,old_price,new_price)
  VALUES(OLD.product_id,OLD.price,NEW.price);
 END IF;
 RETURN NEW;
END;
$$;
```

Trigger:
```sql
CREATE TRIGGER trg_product_price_audit
AFTER UPDATE OF price ON products
FOR EACH ROW EXECUTE FUNCTION log_product_price_change();
UPDATE products SET price=price+25 WHERE product_id=102;
SELECT product_id,old_price,new_price FROM product_price_audit;
```
Expected one audit row for product 102; exact old/new values depend on prior exercises. OLD is before, NEW after; FOR EACH ROW runs per changed row; IS DISTINCT FROM handles NULL safely; RETURN NEW returns changed row in this row-level trigger.

Triggers can hide behavior. Document them and avoid complex cross-system workflows inside them.

## 9. Identity and object inspection

```sql
CREATE TABLE automation_jobs(
 job_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
 job_name VARCHAR(100) NOT NULL,
 created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
INSERT INTO automation_jobs(job_name)
VALUES('daily_customer_extract') RETURNING job_id,job_name;
```
RETURNING displays inserted values. MySQL commonly uses AUTO_INCREMENT. In psql: \df functions, \dt tables, \dv views.

## 10. Automation principles

Good database-local tasks: audit sensitive changes, reusable reporting view, deterministic calculation, local integrity rule. Be cautious with hidden trigger writes, over-privileged procedures, expensive functions, stale materialized views and duplicate logic across Python/SQL. Cross-system orchestration normally belongs in a workflow orchestrator.

## 11. Mini-project — Database Automation Lab

Create order-detail view, order-total materialized view, line-total function, price update procedure, audit table/trigger. Test an update, inspect audit row, document refresh and privileges.

**Deliverables:** SQL setup/test scripts, outputs, object inventory.

## 12. Self-check

Explain view/materialized view, function/procedure, trigger, OLD/NEW, identity, audit logging, trigger risks and orchestration boundary.

## 99. Next lesson

Next: Day 28 — Database Administration Fundamentals.