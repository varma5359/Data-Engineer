# Day 26 Interview Preparation: Database Normalization

**Pathway:** TechPhile.ai — Data Engineer Career Pathway  
**Phase:** 3 — SQL and Database Engineering  
**Level:** Beginner to junior Data Engineer  
**Mini-project:** Data Model Comparison  
**Coverage:** 1NF, 2NF, 3NF, normalization, denormalization, data redundancy, database design trade-offs

---

## 1. How to answer

For every question, define the concept, explain why a Data Engineer uses it, give a TechStore example, describe the result and mention a limitation or common mistake. Avoid memorizing only one-line definitions.

## 2. Core interview questions

### 1. What is normalization?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 2. What is data redundancy?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 3. What is an update anomaly?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 4. What is an insertion anomaly?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 5. What is a deletion anomaly?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 6. What is 1NF?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 7. What is an atomic value?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 8. What is a repeating group?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 9. What is 2NF?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 10. What is a partial dependency?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 11. Why is 2NF relevant to composite keys?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 12. What is 3NF?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 13. What is a transitive dependency?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 14. What is a functional dependency?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 15. What is a candidate key?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 16. Why separate customers and orders?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 17. Why separate products and order items?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 18. Why store unit_price on order_items?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 19. What is denormalization?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

### 20. Why may a warehouse be denormalized?

**Answer guidance:** Explain in plain English, show the smallest relevant SQL/Python example from the class notes, describe the expected behavior and mention one edge case or trade-off.

**Follow-up:** How would you verify this on the TechStore training database?

## 3. Scenario-based discussion

1. **Customer email is repeated in hundreds of order rows. What anomalies can occur?**

   Discuss business impact, inspect source and target state, choose the correct SQL/Python behavior, and explain verification and failure handling.

2. **An order-line table stores order_date and product_name with a composite key. What dependencies exist?**

   Discuss business impact, inspect source and target state, choose the correct SQL/Python behavior, and explain verification and failure handling.

3. **A warehouse requests a wide sales table. Is that always a bad design?**

   Discuss business impact, inspect source and target state, choose the correct SQL/Python behavior, and explain verification and failure handling.

## 4. Mini-project defence — Data Model Comparison

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
