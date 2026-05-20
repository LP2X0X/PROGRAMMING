---
tags: [sql, dml, aggregation]
---

- `GROUP BY` divides rows into groups and applies [[Aggregate Functions]] to each group separately.

---

### Basic Syntax

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department;
```

| department  | employee_count |
| ----------- | -------------- |
| Engineering | 15             |
| Marketing   | 8              |
| Sales       | 12             |

---

### The Golden Rule

```ad-warning
Every column in `SELECT` that is **not** inside an aggregate function **must** appear in `GROUP BY`. Otherwise you get an error (or undefined behavior in older MySQL).
```

```sql
-- WRONG: first_name is not aggregated and not in GROUP BY
SELECT department, first_name, COUNT(*)
FROM employees
GROUP BY department;

-- CORRECT:
SELECT department, COUNT(*)
FROM employees
GROUP BY department;
```

---

### GROUP BY Multiple Columns

```sql
SELECT department, job_title, AVG(salary) AS avg_salary
FROM employees
GROUP BY department, job_title;
```

- This creates a group for each unique **(department, job_title)** combination.

---

### GROUP BY with WHERE

- `WHERE` filters rows **before** grouping:
```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
WHERE hire_date > '2023-01-01'  -- only recent hires
GROUP BY department;
```

---

### Execution Order

- Understanding when `GROUP BY` runs in the query pipeline:

```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

- `WHERE` filters individual rows **before** grouping.
- [[HAVING]] filters groups **after** grouping.
- See [[SQL Syntax Basics]] for the full execution order.

---

### GROUP BY with ORDER BY

```sql
SELECT department, COUNT(*) AS cnt
FROM employees
GROUP BY department
ORDER BY cnt DESC;  -- largest departments first
```
