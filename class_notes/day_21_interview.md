# Day 21 Interview Preparation: SQL Data Extraction

**Pathway:** TechPhile.ai — Data Engineer Career Pathway  
**Phase:** 3 — SQL and Database Engineering  
**Level:** Beginner to junior Data Engineer  
**Mini-project:** Multi-Table Extraction Queries  
**Coverage:** SELECT, WHERE, ORDER BY, LIMIT, DISTINCT, filtering, INNER JOIN, LEFT JOIN, RIGHT JOIN

---

## 1. How to answer

For every question, define the concept, explain why a Data Engineer uses it, give a TechStore example, describe the result and mention a limitation or common mistake. Avoid memorizing only one-line definitions.

## 2. Core interview questions

### 1. What does SELECT return?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 2. Why name columns instead of SELECT *?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 3. What does WHERE filter?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 4. Explain AND versus OR.

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 5. What do IN, BETWEEN and LIKE do?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 6. Why use IS NULL rather than = NULL?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 7. What does DISTINCT do?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 8. Why pair LIMIT with ORDER BY?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 9. Explain INNER JOIN.

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 10. Explain LEFT JOIN.

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 11. Explain RIGHT JOIN.

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 12. What is the ON condition?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 13. Why can a join increase row count?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 14. What is a Cartesian product?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 15. How can WHERE remove unmatched LEFT JOIN rows?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 16. How do you join three tables?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 17. What is keyset pagination?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 18. How do you verify an extract?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

## 3. Scenario-based discussion

1. **A report returns more rows after joining orders to order_items. Is this necessarily a bug?**

   Discuss business impact, inspect source and target state, choose the correct SQL/Python behavior, and explain verification and failure handling.

2. **A customer with no orders disappears from a report. How do you fix it?**

   Discuss business impact, inspect source and target state, choose the correct SQL/Python behavior, and explain verification and failure handling.

3. **A daily top-10 extract changes between runs. What might be missing?**

   Discuss business impact, inspect source and target state, choose the correct SQL/Python behavior, and explain verification and failure handling.

## 4. Mini-project defence — Multi-Table Extraction Queries

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
