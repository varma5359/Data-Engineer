# Day 23 Interview Preparation: SQL Window Functions

**Pathway:** TechPhile.ai — Data Engineer Career Pathway  
**Phase:** 3 — SQL and Database Engineering  
**Level:** Beginner to junior Data Engineer  
**Mini-project:** Historical Trend Query Engine  
**Coverage:** ROW_NUMBER, RANK, DENSE_RANK, LAG, LEAD, PARTITION BY, ORDER BY, analytical queries

---

## 1. How to answer

For every question, define the concept, explain why a Data Engineer uses it, give a TechStore example, describe the result and mention a limitation or common mistake. Avoid memorizing only one-line definitions.

## 2. Core interview questions

### 1. What is a window function?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 2. Window function versus GROUP BY?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 3. What does OVER define?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 4. What does PARTITION BY do?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 5. Does PARTITION BY sort rows?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 6. What does ORDER BY inside OVER control?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 7. Does window ORDER BY sort final output?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 8. What does ROW_NUMBER do?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 9. How does RANK handle ties?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 10. How does DENSE_RANK handle ties?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 11. Compare the three ranking functions.

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 12. Why add a tie-breaker to ROW_NUMBER?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 13. What does LAG return?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 14. What does LEAD return?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 15. Why can LAG return NULL?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 16. How do you compare each customer's previous order?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 17. What is a running total?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 18. What does a window frame define?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 19. How do you filter a window result?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

## 3. Scenario-based discussion

1. **The team needs top three orders per customer. Which window pattern?**

   Discuss business impact, inspect source and target state, choose the correct SQL/Python behavior, and explain verification and failure handling.

2. **Two products tie in sales; equal ranks with no gaps are required. Which function?**

   Discuss business impact, inspect source and target state, choose the correct SQL/Python behavior, and explain verification and failure handling.

3. **The first LAG value is NULL. Is that necessarily an error?**

   Discuss business impact, inspect source and target state, choose the correct SQL/Python behavior, and explain verification and failure handling.

## 4. Mini-project defence — Historical Trend Query Engine

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
