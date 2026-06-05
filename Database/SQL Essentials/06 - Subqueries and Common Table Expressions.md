---
tags: [sql, subqueries, cte]
---

- A **subquery** is a query nested inside another query. It lets you use the result of one query as input to another — computing a value, filtering against a list, or checking for existence.
- A **Common Table Expression (CTE)** achieves similar goals but with cleaner, more readable syntax using the `WITH` clause.
- These tools let you break complex problems into steps, answering questions like "employees earning above the company average" or "customers who placed more than 3 orders last month."

---

## Subquery — A Query Inside a Query

- A subquery is enclosed in parentheses and can appear in `WHERE`, `SELECT`, `FROM`, or `HAVING`.
- The outer query is called the **main query** (or parent query). The inner query is the **subquery**.

### Where Subqueries Can Appear

| Location | What the Subquery Returns | Name |
| --- | --- | --- |
| `WHERE col = (subquery)` | A single value | Scalar subquery |
| `WHERE col IN (subquery)` | A list of values | List subquery |
| `WHERE EXISTS (subquery)` | True/false | Existence subquery |
| `SELECT (subquery) AS col` | A single value per row | Scalar subquery in SELECT |
| `FROM (subquery) AS alias` | A result set (virtual table) | Derived table / inline view |

---

## Scalar Subqueries — Return One Value

- A scalar subquery returns a single value (one row, one column). Use it anywhere you'd use a constant.

```sql
-- Employees earning above the company average
SELECT first_name, last_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

- How it works:
  1. The inner query runs first: `SELECT AVG(salary) FROM employees` returns e.g., `65000`.
  2. The outer query becomes: `WHERE salary > 65000`.

```sql
-- The most recent order
SELECT * FROM orders
WHERE order_date = (SELECT MAX(order_date) FROM orders);
```

```ad-warning
A scalar subquery **must** return exactly one row and one column. If it returns multiple rows, you get an error. If it might return zero rows, it returns `NULL` — which can cause the outer `WHERE` to match nothing.
```

---

## List Subqueries — IN, NOT IN

- Returns a list of values, used with `IN` to filter:

```sql
-- Users who have placed at least one order
SELECT first_name, last_name
FROM users
WHERE id IN (SELECT DISTINCT user_id FROM orders);
```

```sql
-- Products that have never been ordered
SELECT product_name
FROM products
WHERE product_id NOT IN (
    SELECT DISTINCT product_id FROM order_items
);
```

```ad-warning
**`NOT IN` and NULLs: a dangerous trap.** If the subquery returns any NULL values, `NOT IN` returns **no rows at all**. This is because of three-valued logic — `x NOT IN (1, 2, NULL)` evaluates to `UNKNOWN` for every value of `x`.

```sql
-- If any product_id in order_items is NULL, this returns NOTHING:
SELECT product_name FROM products
WHERE product_id NOT IN (SELECT product_id FROM order_items);

-- Safe alternative: use NOT EXISTS
SELECT product_name FROM products p
WHERE NOT EXISTS (
    SELECT 1 FROM order_items oi WHERE oi.product_id = p.product_id
);
```

**Rule**: prefer `NOT EXISTS` over `NOT IN` when the subquery column might contain NULLs.
```

---

## EXISTS / NOT EXISTS — Existence Check

- `EXISTS` returns true if the subquery returns **at least one row**. It doesn't care about what columns or values are returned — only whether rows exist.

```sql
-- Users who have placed at least one order
SELECT u.first_name, u.last_name
FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.user_id = u.id
);
```

- `SELECT 1` is conventional — you could also write `SELECT *` or `SELECT 42`. `EXISTS` only checks for row existence.

```sql
-- Users who have NEVER placed an order
SELECT u.first_name, u.last_name
FROM users u
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.user_id = u.id
);
```

```ad-note
`EXISTS` is often more efficient than `IN` for large datasets. `EXISTS` can stop as soon as it finds the first matching row (short-circuit), while `IN` may need to materialize the full list. The query optimizer sometimes rewrites one into the other, but `EXISTS` is the safer default for large subqueries.
```

---

## Correlated vs Non-Correlated Subqueries

This is one of the most important distinctions to understand.

### Non-Correlated (Independent)

- The subquery does **not** reference the outer query. It runs **once**, produces a result, and the outer query uses that result.

```sql
-- Runs once: compute the average, then filter
SELECT first_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

### Correlated (Dependent)

- The subquery **references a column from the outer query**. It runs **once for each row** in the outer query.

```sql
-- For EACH employee, check if they have any orders
SELECT first_name
FROM employees e
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.employee_id = e.id
);
--                                                  ^^^^
--                       references outer query's "e.id"
```

| | Non-Correlated | Correlated |
| --- | --- | --- |
| References outer query? | No | Yes |
| How many times does it run? | Once | Once per outer row |
| Performance | Generally faster | Can be slow on large tables |
| Example keyword | `IN (SELECT ...)` | `EXISTS (SELECT ... WHERE o.id = e.id)` |

```ad-warning
Correlated subqueries can cause serious performance problems on large tables because they execute the subquery for every row in the outer query. For 100,000 outer rows, the subquery runs 100,000 times. Often a `JOIN` achieves the same result with a single pass. Profile with `EXPLAIN` before using correlated subqueries on large datasets.
```

---

## Subquery in SELECT — Scalar Column

- A subquery in the `SELECT` clause computes a value for each row. It must return exactly one value per row (scalar).

```sql
-- Show each user and how many orders they have
SELECT 
    u.first_name,
    u.last_name,
    (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) AS order_count
FROM users u;
```

- This is a correlated subquery (references `u.id`), so it runs once per user.
- For a small number of users, this is fine. For thousands, a `LEFT JOIN` + `GROUP BY` is typically more efficient:

```sql
-- Same result, often faster
SELECT u.first_name, u.last_name, COUNT(o.order_id) AS order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.first_name, u.last_name;
```

---

## Subquery in FROM — Derived Table

- A subquery in the `FROM` clause creates a temporary result set that you can query like a table. This is called a **derived table** (or **inline view**).

```sql
-- Find departments where the average salary exceeds 60K
SELECT sub.department, sub.avg_salary
FROM (
    SELECT department, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
) AS sub
WHERE sub.avg_salary > 60000;
```

- The derived table **must have an alias** (`AS sub`). SQL Server and MySQL both require this.
- Derived tables are useful when you need to filter on an aggregated value but want to avoid `HAVING`, or when you need to join against a computed result.

```sql
-- Compare each employee's salary to their department average
SELECT 
    e.first_name,
    e.salary,
    dept_avg.avg_salary,
    e.salary - dept_avg.avg_salary AS diff_from_avg
FROM employees e
INNER JOIN (
    SELECT department, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
) AS dept_avg ON e.department = dept_avg.department
ORDER BY diff_from_avg DESC;
```

---

## Common Table Expressions (CTE) — The WITH Clause

- A CTE is a named, temporary result set defined with `WITH`. It makes complex queries far more readable than nested subqueries.

```sql
WITH HighSpenders AS (
    SELECT user_id, SUM(total) AS total_spent
    FROM orders
    GROUP BY user_id
    HAVING SUM(total) > 500
)
SELECT u.first_name, u.last_name, hs.total_spent
FROM users u
INNER JOIN HighSpenders hs ON u.id = hs.user_id
ORDER BY hs.total_spent DESC;
```

### Why CTEs Are Better Than Nested Subqueries

Compare these two equivalent queries:

**Nested subquery — hard to read:**

```sql
SELECT u.first_name, sub.total_spent
FROM users u
INNER JOIN (
    SELECT user_id, SUM(total) AS total_spent
    FROM orders
    GROUP BY user_id
    HAVING SUM(total) > 500
) AS sub ON u.id = sub.user_id;
```

**CTE — much clearer:**

```sql
WITH HighSpenders AS (
    SELECT user_id, SUM(total) AS total_spent
    FROM orders
    GROUP BY user_id
    HAVING SUM(total) > 500
)
SELECT u.first_name, hs.total_spent
FROM users u
INNER JOIN HighSpenders hs ON u.id = hs.user_id;
```

- The logic is defined at the top (the `WITH` block), and the main query at the bottom reads like plain English.

### Multiple CTEs

- You can define multiple CTEs, separated by commas. Later CTEs can reference earlier ones:

```sql
WITH 
OrderTotals AS (
    SELECT user_id, SUM(total) AS total_spent, COUNT(*) AS order_count
    FROM orders
    GROUP BY user_id
),
TopCustomers AS (
    SELECT user_id, total_spent, order_count
    FROM OrderTotals
    WHERE total_spent > 1000
)
SELECT u.first_name, u.last_name, tc.total_spent, tc.order_count
FROM users u
INNER JOIN TopCustomers tc ON u.id = tc.user_id
ORDER BY tc.total_spent DESC;
```

### Referencing a CTE Multiple Times

- Unlike a derived table (which is embedded in one spot), a CTE can be referenced multiple times in the main query:

```sql
WITH DeptStats AS (
    SELECT department, AVG(salary) AS avg_salary, COUNT(*) AS emp_count
    FROM employees
    GROUP BY department
)
SELECT 
    d1.department,
    d1.avg_salary,
    d1.emp_count,
    (SELECT AVG(avg_salary) FROM DeptStats) AS company_avg
FROM DeptStats d1
WHERE d1.avg_salary > (SELECT AVG(avg_salary) FROM DeptStats);
```

```ad-important
**CTEs are not cached or materialized** (in most databases). Each reference to the CTE re-executes its query. If performance matters and the CTE is referenced multiple times, consider using a temporary table instead. SQL Server may internally optimize some CTE references, but don't rely on it.
```

---

## CTE vs Subquery vs Temp Table

| Feature | Subquery | CTE | Temp Table |
| --- | --- | --- | --- |
| Readability | Low (nested) | High (named, at top) | Medium |
| Reusable in same query? | No (must repeat) | Yes (reference by name) | Yes |
| Materialized (stored)? | No | No (re-executed) | Yes (on disk) |
| Scope | Single use | Single statement | Session or explicit drop |
| Performance | Depends | Same as subquery | Better for large, reused datasets |

---

## Recursive CTEs

- A CTE can reference itself, creating a **recursive query**. Useful for hierarchical data (org charts, category trees, bill of materials).

```sql
-- Employee hierarchy: find all reports under a manager
WITH RECURSIVE EmployeeTree AS (
    -- Anchor: start with the top-level manager
    SELECT employee_id, first_name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive: find employees who report to someone already in the tree
    SELECT e.employee_id, e.first_name, e.manager_id, et.level + 1
    FROM employees e
    INNER JOIN EmployeeTree et ON e.manager_id = et.employee_id
)
SELECT * FROM EmployeeTree ORDER BY level, first_name;
```

```ad-note
**SQL Server** does not use the `RECURSIVE` keyword — just `WITH`:

```sql
WITH EmployeeTree AS (
    SELECT employee_id, first_name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    SELECT e.employee_id, e.first_name, e.manager_id, et.level + 1
    FROM employees e
    INNER JOIN EmployeeTree et ON e.manager_id = et.employee_id
)
SELECT * FROM EmployeeTree ORDER BY level, first_name;
```

**MySQL/MariaDB** requires `WITH RECURSIVE` (following the SQL standard).
```

```ad-warning
Recursive CTEs can loop infinitely if the data has cycles (e.g., employee A reports to B, B reports to A). Add a `MAXRECURSION` option (SQL Server) or a level limit in the `WHERE` clause to protect against this:

```sql
-- SQL Server: limit to 100 levels
OPTION (MAXRECURSION 100);

-- Any database: stop at level 20
WHERE et.level < 20
```
```

---

## When to Use Subquery vs JOIN

| Situation | Prefer | Why |
| --- | --- | --- |
| Filter against a list of values | `IN (subquery)` or `EXISTS` | Clean and readable |
| Compare to a single computed value | Scalar subquery | Natural fit |
| Combine columns from both tables | `JOIN` | Subquery can't add columns from the inner table easily |
| Check for existence/absence | `EXISTS` / `NOT EXISTS` | More efficient than `IN`, NULL-safe |
| Large inner result set | `JOIN` | JOINs are generally optimized better for large sets |
| Complex multi-step logic | `CTE` | Break it into named steps for readability |

```ad-note
In practice, modern query optimizers often generate the same execution plan for `JOIN`, `IN`, and `EXISTS` variations. Write for readability first, then profile if performance matters. When in doubt, `JOIN` for combining data, `EXISTS` for checking existence, CTE for multi-step logic.
```

---

## Practical Examples

### Employees in the Highest-Paid Department

```sql
WITH DeptSalaries AS (
    SELECT department, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
)
SELECT e.first_name, e.last_name, e.department, e.salary
FROM employees e
INNER JOIN DeptSalaries ds ON e.department = ds.department
WHERE ds.avg_salary = (SELECT MAX(avg_salary) FROM DeptSalaries);
```

### Customers Who Ordered Every Product

```sql
-- Customers whose distinct product count equals the total number of products
SELECT c.customer_name
FROM customers c
WHERE (
    SELECT COUNT(DISTINCT oi.product_id)
    FROM orders o
    INNER JOIN order_items oi ON o.order_id = oi.order_id
    WHERE o.customer_id = c.customer_id
) = (SELECT COUNT(*) FROM products);
```

### Running Total Using a Correlated Subquery

```sql
-- Running total of order amounts (correlated subquery approach)
SELECT 
    order_id,
    order_date,
    total,
    (SELECT SUM(o2.total) 
     FROM orders o2 
     WHERE o2.order_date <= o1.order_date) AS running_total
FROM orders o1
ORDER BY order_date;
```

```ad-note
The running total example is better solved with [[Window Functions Overview|window functions]] (`SUM(total) OVER (ORDER BY order_date)`) which are covered in the SQL folder under Advanced Queries. The correlated subquery version is shown here for learning purposes but is much slower.
```

```ad-note
With subqueries and CTEs you can express almost any question in SQL. For reusable, named queries and server-side logic, see [[07 - Views and Stored Procedures]].
```
