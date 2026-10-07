---
tags: [sql, subqueries, advanced]
---

- A **subquery** (also called an inner query or nested query) is a query embedded inside another query. The outer query uses the subquery's result.

```ad-note
Like any query, a subquery returns a result set that may consist of:
- A single row with a single column 
- Multiple rows with a single column 
- Multiple rows having multiple columns 
  
The type of result set returned by the subquery determines how it may be used and which operators the containing statement may use to interact with the data the subquery returns.
```

### Subquery Result Types

Like any query, a subquery returns a result set. The **shape** of that result determines where you can use it and which operators work with it:

| Result Type | Shape | Where You Can Use It | Operators |
|---|---|---|---|
| **Scalar** | 1 row, 1 column | `SELECT`, `WHERE`, anywhere a single value is expected | `=`, `<>`, `<`, `>`, `<=`, `>=` |
| **Column** | Multiple rows, 1 column | `WHERE` — compare against a list of values | `IN`, `NOT IN`, `ANY`, `ALL`, `EXISTS` |
| **Table** | Multiple rows, multiple columns | `FROM` — use as a derived table | Treated as a table, so you query it with `SELECT` |

**Scalar** — returns one value, acts like a literal:
```sql
-- "What is the average salary?" → one number
WHERE salary > (SELECT AVG(salary) FROM employees)
--              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ returns e.g. 65000
```

**Column** — returns a list of values, acts like a set:
```sql
-- "Which departments are in New York?" → a list of IDs
WHERE department_id IN (SELECT id FROM departments WHERE location = 'NY')
--                      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ returns e.g. (1, 4, 7)
```

**Table** — returns rows and columns, acts like a temporary table:
```sql
-- "What are the stats per department?" → a full result set
FROM (SELECT department, AVG(salary) AS avg_sal FROM employees GROUP BY department) AS stats
--   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ returns rows with dept + avg_sal columns
```

```ad-warning
If a subquery returns **multiple rows** but you use it with a scalar operator like `=`, you get an error. The operator must match the shape of the result.

```sql
-- ERROR: subquery returns more than one row
WHERE salary = (SELECT salary FROM employees WHERE department = 'Sales')
-- Use IN instead:
WHERE salary IN (SELECT salary FROM employees WHERE department = 'Sales')
```

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
