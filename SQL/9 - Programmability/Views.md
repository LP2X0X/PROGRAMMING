---
tags: [sql, programmability]
---

- A **view** is a named, saved query that acts like a virtual table. It doesn't store data itself — it runs the underlying query every time you access it.

---

### Creating a View

```sql
CREATE VIEW active_employees AS
SELECT id, first_name, last_name, department, salary
FROM employees
WHERE active = TRUE;
```

- Now you can query it like a table:
```sql
SELECT * FROM active_employees WHERE department = 'Engineering';
```

---

### CREATE OR REPLACE

- Update an existing view without dropping it first:
```sql
CREATE OR REPLACE VIEW active_employees AS
SELECT id, first_name, last_name, department, salary, hire_date
FROM employees
WHERE active = TRUE;
```

---

### Dropping a View

```sql
DROP VIEW active_employees;
DROP VIEW IF EXISTS active_employees;
```

---

### Benefits

- **Simplify complex queries**: wrap a multi-join query in a view, then query the view with simple SELECTs.
- **Abstraction layer**: hide table complexity from users or applications. If the underlying schema changes, update the view instead of every query.
- **Security**: expose only certain columns to certain users. Grant SELECT on the view without granting access to the base table.
- **Reusability**: define the logic once, use it in many queries.

---

### Updatable Views

- Simple views (single table, no aggregates, no DISTINCT, no GROUP BY) can support `INSERT`, `UPDATE`, `DELETE`:
```sql
UPDATE active_employees SET salary = 90000 WHERE id = 42;
-- This modifies the underlying employees table
```

- Complex views (joins, aggregates, subqueries) are generally **read-only**.

---

### Materialized Views

- A **materialized view** physically stores the query result on disk (unlike regular views):
```sql
-- PostgreSQL:
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT DATE_TRUNC('month', order_date) AS month, SUM(total) AS total_sales
FROM orders
GROUP BY 1;

-- Refresh when the data changes:
REFRESH MATERIALIZED VIEW monthly_sales;
```

- Supported in PostgreSQL, Oracle, SQL Server (as "indexed views"). **Not** in MySQL.
- Benefits: much faster reads (pre-computed). Trade-off: data can be stale until refreshed.

```ad-note
Regular views do **not** improve query performance — the underlying query runs every time. If performance is the goal, consider materialized views, [[What are Indexes|indexes]], or caching at the application level.
```
