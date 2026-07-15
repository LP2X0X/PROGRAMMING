---
tags: [sql, subqueries, advanced]
---

- A **subquery** (also called an inner query or nested query) is a query embedded inside another query. The outer query uses the subquery's result.

---

### Subquery in WHERE

- **Scalar subquery** — returns a single value:
```sql
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

- **List subquery** — returns multiple values (use with `IN`):
```sql
SELECT name
FROM employees
WHERE department_id IN (
    SELECT id FROM departments WHERE location = 'New York'
);
```

- **EXISTS / NOT EXISTS** — checks if the subquery returns any rows:
```sql
SELECT c.name
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.id
);
-- Returns customers who have at least one order
```

```ad-tip
`EXISTS` is often faster than `IN` for large datasets because it stops as soon as it finds a single match. See [[Correlated Subqueries]] for how `EXISTS` works.
```

---

### Subquery in FROM (Derived Table)

- The subquery result acts as a temporary table that you can query:
```sql
SELECT dept, avg_salary
FROM (
    SELECT department AS dept, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
) AS dept_stats
WHERE avg_salary > 70000;
```

- The alias (`AS dept_stats`) is **required** for derived tables.

---

### Subquery in SELECT (Scalar)

- Returns one value per row of the outer query:
```sql
SELECT 
    e.name,
    e.salary,
    (SELECT AVG(salary) FROM employees) AS company_avg
FROM employees e;
```

```ad-warning
Scalar subqueries in `SELECT` run once per row of the outer query. For large [[Result Set|result sets]] this can be very slow. Consider using [[Common Table Expressions]] or [[INNER JOIN]] instead.
```

---

### Subqueries vs JOINs

| Use a Subquery when...                        | Use a JOIN when...                          |
| --------------------------------------------- | ------------------------------------------- |
| You only need data from the outer table       | You need columns from both tables           |
| Checking existence (`EXISTS`, `IN`)           | Combining data from multiple tables         |
| The logic is naturally hierarchical/nested    | Performance matters for large datasets      |

- Most subqueries can be rewritten as JOINs and vice versa. The optimizer often produces the same execution plan.
- [[Common Table Expressions]] are a more readable alternative to deeply nested subqueries.
