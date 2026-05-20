---
tags: [sql, dml, aggregation]
---

- **Aggregate functions** compute a single result from a set of rows. They collapse multiple rows into one summary value.

---

### The Big Five

| Function                | Description                           |
| ----------------------- | ------------------------------------- |
| `COUNT(*)`              | Number of rows (including NULLs)      |
| `COUNT(column)`         | Number of non-NULL values in column   |
| `COUNT(DISTINCT column)` | Number of distinct non-NULL values   |
| `SUM(column)`           | Total of all values (numeric only)    |
| `AVG(column)`           | Mean of all values (numeric only)     |
| `MIN(column)`           | Smallest value (works on all types)   |
| `MAX(column)`           | Largest value (works on all types)    |

---

### Examples

```sql
SELECT 
    COUNT(*) AS total_employees,
    COUNT(manager_id) AS has_manager,
    COUNT(DISTINCT department) AS departments,
    SUM(salary) AS total_payroll,
    AVG(salary) AS avg_salary,
    MIN(salary) AS lowest_salary,
    MAX(salary) AS highest_salary
FROM employees;
```

---

### NULL Handling

```ad-note
All aggregate functions **ignore NULL values**, except `COUNT(*)` which counts rows regardless of NULLs.
```

- This affects `AVG` in particular:
```sql
-- If salaries are: 100, 200, NULL
-- AVG = (100 + 200) / 2 = 150  (not 100 — NULL is skipped, not treated as 0)
-- COUNT(*) = 3, COUNT(salary) = 2
```

---

### Without GROUP BY

- When used without [[GROUP BY]], aggregate functions treat the **entire table** as a single group:
```sql
SELECT AVG(salary) FROM employees;
-- Returns one row: the average of all salaries
```

---

### With DISTINCT

```sql
-- Count unique departments
SELECT COUNT(DISTINCT department) FROM employees;

-- Sum of unique salary values only
SELECT SUM(DISTINCT salary) FROM employees;
```

```ad-tip
`MIN` and `MAX` work on strings (alphabetical order) and dates too — not just numbers. `MIN(name)` returns the alphabetically first name, and `MAX(hire_date)` returns the most recent date.
```
