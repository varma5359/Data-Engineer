# Day 27 Interview Preparation: Database Objects and Automation

**Pathway:** TechPhile.ai — Data Engineer Career Pathway  
**Phase:** 3 — SQL and Database Engineering  
**Level:** Beginner to junior Data Engineer  
**Mini-project:** Database Automation Lab  
**Coverage:** Views, stored procedures, triggers, functions, database automation concepts

---

## 1. How to answer

For every question, define the concept, explain why a Data Engineer uses it, give a TechStore example, describe the result and mention a limitation or common mistake. Avoid memorizing only one-line definitions.

## 2. Core interview questions

### 1. What is a database view?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 2. Does a normal view store result rows?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 3. What is a materialized view?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 4. Why refresh it?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 5. Are all views updatable?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 6. What is a database function?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 7. How do you call a PostgreSQL function?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 8. What is a procedure?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 9. How do you call a PostgreSQL procedure?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 10. Function versus procedure?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 11. What is a trigger?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 12. What are trigger events?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 13. What are OLD and NEW?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 14. What is a row-level trigger?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 15. What is an audit trigger?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 16. What risks do triggers introduce?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 17. What is an identity column?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 18. When should logic live in the database?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 19. When should orchestration live outside it?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

## 3. Scenario-based discussion

1. **Five teams repeat the same reporting query. How might a view help?**

   Discuss business impact, inspect source and target state, choose the correct SQL/Python behavior, and explain verification and failure handling.

2. **A materialized view is stale. What process is missing?**

   Discuss business impact, inspect source and target state, choose the correct SQL/Python behavior, and explain verification and failure handling.

3. **A team wants a cross-system workflow inside a trigger. What would you recommend?**

   Discuss business impact, inspect source and target state, choose the correct SQL/Python behavior, and explain verification and failure handling.

## 4. Mini-project defence — Database Automation Lab

Prepare a 2–3 minute explanation:
- What business problem did it solve?
- Which tables or components did you use?
- Why did you choose this approach?
- What output did you verify?
- Which edge case or failure did you test?
- What would you improve before production?

## 5. Common interview mistakes

- Reciting syntax without explaining the result.
- Ignoring NULL values, duplicate rows and one-to-many relationships.
- Claiming performance improved without measuring it.
- Ignoring transaction boundaries and error handling.
- Using administrator privileges for routine jobs.
- Exposing credentials in code or logs.
- Failing to verify row counts and business totals.

## 6. Readiness checklist

- [ ] I can explain every key concept in today's syllabus.
- [ ] I can write or interpret the main SQL/Python pattern.
- [ ] I can explain the expected output.
- [ ] I can identify common mistakes and trade-offs.
- [ ] I can defend the mini-project.
- [ ] I can explain production validation and failure handling.
