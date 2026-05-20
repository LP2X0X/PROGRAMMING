---
tags: [sql, subqueries, advanced]
---

- A **correlated subquery** references columns from the **outer query**. Unlike a regular subquery (which runs once), a correlated subquery is re-evaluated for **each row** of the outer query.

---

### Basic Example

```sql
-- Find employees who earn more than their department's average
SELECT e.name, e.salary, e.department
FROM employees e
WHERE e.salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE department = e.department  -- references outer query's row
);
```

- For each employee `e`, the subquery calculates the average salary of **that employee's department**, then compares.

---

### EXISTS with Correlated Subquery

- The most common use case — check if a related row exists:
```sql
-- Customers who have placed at least one order
SELECT c.name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id  -- correlated to outer row
);
```

- `NOT EXISTS` is the inverse — find customers with **no** orders:
```sql
SELECT c.name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

```ad-tip
`EXISTS` only checks for the presence of rows — `SELECT 1` is convention since the actual selected values don't matter. The database stops as soon as it finds one matching row.
```

---

### Performance Considerations

```ad-warning
Correlated subqueries can be slow on large datasets because they re-execute for each row of the outer query. If you see performance issues, consider rewriting as a `JOIN`:
```

```sql
-- Correlated subquery (slower):
SELECT e.name, e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(salary) FROM employees WHERE department = e.department
);

-- Equivalent JOIN (often faster):
SELECT e.name, e.salary
FROM employees e
INNER JOIN (
    SELECT department, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
) dept_avg ON e.department = dept_avg.department
WHERE e.salary > dept_avg.avg_salary;
```

- Modern query optimizers can sometimes transform correlated subqueries into joins automatically, but not always.
- For the `EXISTS` pattern specifically, performance is usually good because the database can stop early. The concern is mainly with correlated scalar subqueries.
