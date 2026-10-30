---
tags: [sql, window-functions, advanced]
---

- Ranking functions assign a number to each row based on its position within a partition. They differ in how they handle **ties** (rows with equal values).

---

### Comparison

Given employees ordered by salary DESC within each department:

| Function       | Behavior with ties                   | Example output  |
| -------------- | ------------------------------------ | --------------- |
| `ROW_NUMBER()` | No ties — always unique, arbitrary   | 1, 2, 3, 4, 5  |
| `RANK()`       | Same rank for ties, **gaps** after   | 1, 2, 2, **4**, 5 |
| `DENSE_RANK()` | Same rank for ties, **no gaps**      | 1, 2, 2, **3**, 4 |

```sql
SELECT 
    name, department, salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS row_num,
    RANK()       OVER (PARTITION BY department ORDER BY salary DESC) AS rnk,
    DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dense_rnk
FROM employees;
```

---

### Top-N Per Group Pattern

- One of the most commonly used window function patterns — find the top N rows within each group:
```sql
WITH ranked AS (
    SELECT 
        name, department, salary,
        ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn
    FROM employees
)
SELECT name, department, salary
FROM ranked
WHERE rn <= 3;
-- Top 3 highest-paid employees per department
```

```ad-note
You cannot use window functions directly in `WHERE` — they're evaluated after `WHERE` in the execution order. Wrap in a [[Common Table Expressions|CTE]] or subquery and filter on the alias. See [[SQL Syntax Basics]] for execution order.
```

---

### NTILE

- Divides rows into `n` roughly equal groups and assigns a group number:
```sql
SELECT 
    name, salary,
    NTILE(4) OVER (ORDER BY salary) AS quartile
FROM employees;
-- Assigns each employee to salary quartile 1-4
```

- Useful for percentile calculations and bucketing.

---

### When to Use Which

| Need                              | Use             |
| --------------------------------- | --------------- |
| Unique sequential number          | `ROW_NUMBER()`  |
| Rank with gaps (competition-style)| `RANK()`        |
| Rank without gaps                 | `DENSE_RANK()`  |
| Top N per group                   | `ROW_NUMBER()` (most common) |
| Split into equal buckets          | `NTILE(n)`      |
