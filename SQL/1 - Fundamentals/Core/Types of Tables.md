---
tags:
  - sql
  - fundamentals
---
# Types of Tables

- SQL recognizes four different types of tables that can appear in a query's `FROM` clause:

| Type | Created With | Storage | Lifetime |
|---|---|---|---|
| **Permanent** | `CREATE TABLE` | On disk | Until explicitly dropped |
| **Derived** | Subquery in `FROM` | In memory | Duration of the query |
| **Temporary** | `CREATE TEMPORARY TABLE` | In memory (or disk) | Until session ends |
| **Virtual** | `CREATE VIEW` | Not stored (query re-runs each time) | Until explicitly dropped |

## Permanent Tables

- Standard tables created with `CREATE TABLE` that persist on disk.
- This is what most people mean when they say "table."

## Derived Tables

- A subquery in the `FROM` clause produces a result set that the outer query treats as a table:

```sql
SELECT dept, avg_sal
FROM (
    SELECT department AS dept, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department
) AS dept_stats
WHERE avg_sal > 50000;
```

- The alias (`AS dept_stats`) is **required** — the server needs a name to reference columns.
- Exists only in memory for the duration of the surrounding query.

## Temporary Tables

- Created with `CREATE TEMPORARY TABLE` and scoped to the current session:

```sql
CREATE TEMPORARY TABLE temp_results (
    id INT,
    value DECIMAL(10,2)
);
```

- Automatically dropped when the session ends.
- Only visible to the session that created them.

## Virtual Tables (Views)

- A **view** is a named, saved query that behaves like a table but stores no data:

```sql
CREATE VIEW active_customers AS
SELECT * FROM customers WHERE status = 'active';
```

- Every time you query the view, the underlying `SELECT` runs again.
- Useful for simplifying complex queries, restricting access, or providing a stable interface over changing schemas.

## See Also

- [[CREATE TABLE]]
- [[Subqueries]]
- [[Views]]
