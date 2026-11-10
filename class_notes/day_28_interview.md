# Day 28 Interview Preparation: Database Administration Fundamentals

**Pathway:** TechPhile.ai — Data Engineer Career Pathway  
**Phase:** 3 — SQL and Database Engineering  
**Level:** Beginner to junior Data Engineer  
**Mini-project:** Database Operations Checklist  
**Coverage:** MySQL and PostgreSQL administration, users, roles, permissions, backup, restore, connection management

---

## 1. How to answer

For every question, define the concept, explain why a Data Engineer uses it, give a TechStore example, describe the result and mention a limitation or common mistake. Avoid memorizing only one-line definitions.

## 2. Core interview questions

### 1. What does database administration cover?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 2. What is a database role?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 3. What does LOGIN mean in PostgreSQL?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 4. What is least privilege?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 5. Why not run ETL as superuser?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 6. What does GRANT do?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 7. What does REVOKE do?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 8. What does USAGE on a schema allow?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 9. How can role membership affect privileges?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 10. What is a logical backup?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 11. What is a physical backup?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 12. What does pg_dump do?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 13. What does pg_restore do?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 14. What does mysqldump do?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 15. Why test a restore?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 16. What is point-in-time recovery?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 17. What is RPO?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 18. What is RTO?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 19. What details are needed for a connection?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 20. Why use connection pooling?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

## 3. Scenario-based discussion

1. **A pipeline account has superuser access. What would you change?**

   Discuss business impact, inspect source and target state, choose the correct SQL/Python behavior, and explain verification and failure handling.

2. **A backup succeeds but restore fails. What lesson?**

   Discuss business impact, inspect source and target state, choose the correct SQL/Python behavior, and explain verification and failure handling.

3. **A service cannot connect after deployment. What do you check?**

   Discuss business impact, inspect source and target state, choose the correct SQL/Python behavior, and explain verification and failure handling.

## 4. Mini-project defence — Database Operations Checklist

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
