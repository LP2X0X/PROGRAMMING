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

---

### WITH ROLLUP

Adds **subtotal and grand total rows** automatically to a `GROUP BY` result:

```sql
SELECT department, job_title, SUM(salary)
FROM employees
GROUP BY department, job_title WITH ROLLUP;
```

| department  | job_title | SUM    |
| ----------- | --------- | ------ |
| Engineering | Junior    | 60000  |
| Engineering | Senior    | 90000  |
| Engineering | NULL      | 150000 |
| Sales       | Junior    | 40000  |
| Sales       | Senior    | 70000  |
| Sales       | NULL      | 110000 |
| NULL        | NULL      | 260000 |

- `NULL` in the rollup rows means "all values" for that column
- Rolls up **right to left** through `GROUP BY` columns:
	1. Groups by `department + job_title` (normal detail rows)
	2. Groups by `department` only (subtotal per department)
	3. Groups by nothing (grand total)
- Think of it like Excel pivot table subtotals — detail + summary in one query
