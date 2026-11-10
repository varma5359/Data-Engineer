# Day 27 Interview Preparation: Database Objects and Automation

**Phase:** 3 — SQL and Database Engineering  
**Level:** Beginner to junior Data Engineer  
**Mini-project:** Database Automation Lab  
**Database:** PostgreSQL

---

## 1. Core questions with complete answers

### 1. What are database objects?

Database objects are named structures stored in a database schema. Examples include tables, views, materialized views, indexes, functions, procedures, triggers and sequences. Each object serves a different purpose in storing, querying, securing or automating data.

### 2. What is a view?

A view is a named query that can be queried like a table for many SELECT operations. It can simplify repeated joins, provide a reusable interface and expose selected columns to users.

### 3. Does a normal PostgreSQL view store data?

A normal view stores the query definition, not a separate copy of its result rows. When queried, it reads the underlying data according to the current transaction's visibility rules.

### 4. What is a materialized view?

A materialized view stores the result of a query physically. It can make repeated analytical reads faster, but its result may become stale until refreshed.

### 5. View versus materialized view?

A normal view stores a query definition and reflects current underlying data when queried. A materialized view stores a result snapshot and needs a refresh to reflect later source changes.

### 6. What is REFRESH MATERIALIZED VIEW?

In PostgreSQL it recomputes the materialized view's stored result from its defining query. It is used when source data has changed and the materialized result needs updating.

### 7. What is a database function?

A function is a named reusable routine that accepts parameters and returns a value or a table. It can encapsulate a calculation or query logic.

### 8. What is a stored procedure?

A stored procedure is a named database routine invoked as an operation. In PostgreSQL it is called using CALL. It is commonly used to execute a sequence of database actions.

### 9. Function versus procedure in PostgreSQL?

Functions declare a return type and are commonly used in expressions or SELECT statements. Procedures are invoked with CALL and have different transaction-control capabilities. Exact behavior and syntax vary across database products.

### 10. Can a function accept parameters?

Yes. A function can accept input parameters with declared types. For example, calculate_line_total(quantity INTEGER, unit_price NUMERIC) can return the product of those values.

### 11. What is PL/pgSQL?

PL/pgSQL is PostgreSQL's procedural language. It supports variables, conditions, loops, exceptions and SQL statements inside database functions and procedures.

### 12. What is a trigger?

A trigger is database logic that runs automatically in response to a specified event on a table or view, such as INSERT, UPDATE or DELETE.

### 13. What is a trigger function in PostgreSQL?

A PostgreSQL trigger executes a function that returns the special TRIGGER type. The trigger definition specifies the event, timing, table and whether it runs per row or per statement.

### 14. What is the difference between BEFORE and AFTER triggers?

A BEFORE trigger runs before the row operation and can validate or modify NEW. An AFTER trigger runs after the row operation and is useful for actions such as audit inserts. Both participate in the surrounding transaction.

### 15. What are OLD and NEW in a trigger?

OLD represents the row before the operation. NEW represents the new row for INSERT or UPDATE. Availability depends on the trigger event; for DELETE, OLD is available and NEW is not.

### 16. What does FOR EACH ROW mean?

It means the trigger fires once for every affected row. A statement-level trigger runs once for the SQL statement, regardless of the number of rows affected.

### 17. What is an audit trigger?

An audit trigger automatically records information about a data change, such as the affected key, old value, new value and timestamp. It supports traceability, but audit access and retention must be secured.

### 18. Does an audit trigger run in a separate transaction?

No. A trigger's database writes normally participate in the same transaction as the statement that fired it. If that transaction rolls back, the trigger's audit insert rolls back too.

### 19. What happens if a trigger function raises an error?

The triggering statement fails. If it is inside a transaction, the transaction may enter an aborted state and need rollback or rollback to a savepoint, depending on the context.

### 20. Can a trigger cause recursion?

Yes. A trigger that updates the same table or another table with triggers can cause repeated trigger execution. Design and test trigger logic carefully to prevent recursion and unexpected side effects.

### 21. Why should triggers be used carefully?

Triggers hide work behind ordinary table statements. They can affect performance, complicate debugging and create unexpected side effects. Use them when a database-level guarantee is important and document their behavior.

### 22. What is a sequence?

A sequence is a database object that generates numeric values, often used for identifiers. PostgreSQL identity columns use sequence-backed generation.

### 23. What is a schema?

A schema is a namespace within a database that organizes objects and helps manage naming and permissions. For example, automation_lab.orders identifies the orders table in the automation_lab schema.

### 24. What is a database index?

An index is a data structure that can help locate rows efficiently. It is covered in detail in Day 25.

### 25. Can a view improve security?

A view can expose only selected columns or rows, and privileges can be granted on the view rather than directly on base tables. However, security behavior depends on ownership, privileges, security options and database configuration. A view alone is not automatically a complete security boundary.

### 26. What is an updatable view?

Some simple views allow INSERT, UPDATE or DELETE operations to affect underlying tables. Complex views with joins, grouping or aggregates may not be directly updatable without additional mechanisms. Check the database's rules before relying on updates through a view.

### 27. What is the difference between a view and a table?

A table stores data rows. A normal view stores a query definition and presents a result derived from underlying objects. A materialized view stores a query result.

### 28. How do you inspect database objects?

Use database catalog views, information_schema views, or client tools such as pgAdmin and psql. Inspect object definitions and dependencies before changing or dropping objects.

### 29. What is CREATE OR REPLACE used for?

In PostgreSQL it replaces the definition of a supported existing object, such as a function or view, while preserving the object identity where supported. It does not mean all dependent objects or interfaces are automatically safe; review compatibility before replacing production definitions.

### 30. How would you automate a daily summary?

I could create a query or view for reusable logic, use a procedure or scheduled orchestration job to refresh or populate a summary, and validate row counts and totals. The scheduler and retry strategy should be designed explicitly rather than assuming a database object schedules itself.

---

## 2. Scenario-based interview questions

### Scenario 1: Analysts repeat the same multi-table query

**Good answer:** I would consider creating a view that exposes the required columns and joins. This provides a reusable interface. I would document its grain and permissions, and inspect the query plan if it becomes slow.

### Scenario 2: A dashboard repeatedly calculates the same expensive summary

**Good answer:** I would measure the query and consider a materialized view or summary table. I would define its refresh frequency, freshness requirement, failure handling and reconciliation checks.

### Scenario 3: A status change must be audited

**Good answer:** A database trigger can write old and new status values to an audit table. I would ensure the audit table is protected, test multi-row updates and understand that audit writes roll back with the original transaction.

### Scenario 4: A trigger slows down a bulk load

**Good answer:** A row-level trigger may execute once per inserted or updated row. I would measure its work, determine whether it is required during the load, and consider staging plus controlled audit processing without weakening required integrity or audit guarantees.

### Scenario 5: A function is called from a SELECT statement

**Good answer:** In PostgreSQL, a function that returns a compatible value can be used in an expression. A procedure is invoked with CALL and is not used as a scalar expression.

### Scenario 6: A procedure attempts to COMMIT but errors

**Good answer:** PostgreSQL procedure transaction control is allowed only in specific CALL contexts and is restricted by how the procedure is invoked. I would check the execution context and avoid assuming a procedure can commit in every situation.

---

## 3. Mini-project defence

Be ready to explain:

1. What problem did the view solve?
2. What is stored in a normal view?
3. Why did the materialized view need refresh?
4. What parameters did your function accept?
5. What did your procedure update?
6. What event fired the trigger?
7. What do OLD and NEW mean?
8. Why did a no-op status update not create an audit record?
9. What happens to the audit row if the transaction rolls back?
10. What are the risks of hidden trigger logic?

---

## 4. Common interview traps

- Saying a normal view stores rows.
- Saying materialized views automatically refresh.
- Calling a procedure as a scalar function.
- Forgetting PostgreSQL trigger functions return TRIGGER.
- Confusing OLD and NEW.
- Saying AFTER trigger writes survive rollback.
- Assuming triggers fire only once for a multi-row statement.
- Ignoring recursion and bulk-load impact.
- Assuming view privileges automatically solve every security problem.
- Assuming PostgreSQL and MySQL routine syntax is identical.

---

## 5. Readiness checklist

- [ ] I can explain tables, views and materialized views.
- [ ] I can create and query a view.
- [ ] I understand materialized-view refresh.
- [ ] I can create and call a simple function.
- [ ] I can create and call a procedure.
- [ ] I can explain function versus procedure.
- [ ] I can create a PostgreSQL trigger function and trigger.
- [ ] I understand OLD and NEW.
- [ ] I understand trigger transaction behavior.
- [ ] I can explain why triggers need careful testing.
