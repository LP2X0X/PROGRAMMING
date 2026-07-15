---
tags: [sql, subqueries, advanced]
---

- A **Common Table Expression (CTE)** is a named temporary [[Result Set|result set]] defined with the `WITH` keyword. It exists only for the duration of the query.

---

### Basic Syntax

```sql
WITH high_earners AS (
    SELECT name, department, salary
    FROM employees
    WHERE salary > 80000
)
SELECT department, COUNT(*) AS count
FROM high_earners
GROUP BY department;
```

- The CTE (`high_earners`) is defined first, then used in the main query like a regular table.

---

### Multiple CTEs

```sql
WITH 
dept_stats AS (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
),
high_paying_depts AS (
    SELECT department_id
    FROM dept_stats
    WHERE avg_salary > 75000
)
SELECT e.name, e.salary, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.id
WHERE e.department_id IN (SELECT department_id FROM high_paying_depts);
```

- Later CTEs can reference earlier ones in the same `WITH` clause.

---

### Recursive CTEs

- Used for hierarchical data (org charts, category trees) and sequences:
```sql
WITH RECURSIVE org_chart AS (
    -- Base case: top-level employees (no manager)
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: join employees to their manager in the CTE
    SELECT e.id, e.name, e.manager_id, oc.level + 1
    FROM employees e
    INNER JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT * FROM org_chart ORDER BY level, name;
```

- SQL Server uses `WITH cte AS (...)` without the `RECURSIVE` keyword — it detects recursion automatically.

```ad-note
Recursive CTEs need a termination condition. The recursion stops when the recursive member returns no new rows. Add a `WHERE level < 10` safety check to prevent infinite loops with circular references.
```

---

### CTE vs Subquery vs Temp Table

| Approach         | Readability | Reusable in query? | Materialized? | Persists after query? |
| ---------------- | ----------- | ------------------- | ------------- | --------------------- |
| CTE              | Best        | Yes (multiple refs) | Usually no    | No                    |
| Subquery         | Okay        | No (inline only)    | No            | No                    |
| Temporary table  | Okay        | Yes                 | Yes (on disk) | Until session ends    |

- CTEs are **not materialized** by default — the optimizer may inline them like subqueries. If you reference a CTE multiple times and performance matters, a temp table might be better.
- Use CTEs primarily for **readability** — they flatten deeply nested logic into a step-by-step pipeline.
