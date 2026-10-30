---
tags: [sql, programmability]
---

- A **view** is a query stored in the **data dictionary** (where the server keeps all metadata about your database). It acts like a virtual table — you can `SELECT` from it like any table — but there is no data associated with it.
- When you query a view, the server **merges** your query with the view's definition to produce a single final query that runs against the real tables:

```sql
-- View definition:
CREATE VIEW active_employees AS
SELECT id, name, email FROM employees WHERE active = TRUE;

-- Your query:
SELECT name FROM active_employees WHERE name LIKE 'A%';

-- What actually executes (merged):
SELECT name FROM employees WHERE active = TRUE AND name LIKE 'A%';
```

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

- A view is updatable (supports `INSERT`, `UPDATE`, `DELETE` that modify the underlying table) if **all** of the following are true:

| Rule | Why |
|---|---|
| No aggregate functions (`MAX`, `MIN`, `AVG`, `SUM`, `COUNT`, etc.) | Aggregates collapse multiple rows into one — which row would you update? |
| No `GROUP BY` or `HAVING` | Same reason — grouped results don't map back to individual rows |
| No subqueries in the `SELECT` or `FROM` clause | Computed/derived columns can't be written back |
| Any subquery in `WHERE` does not reference tables in the `FROM` clause | Would create a circular dependency |
| No `UNION`, `UNION ALL`, or `DISTINCT` | These merge/deduplicate rows — can't trace back to a single source row |
| `FROM` includes at least one table or updatable view | Must have something concrete to write to |
| Only `INNER JOIN` if multiple tables are in `FROM` | Outer joins introduce NULLs for missing rows — which table do you insert into? |

```sql
-- ✅ Updatable — single table, no aggregates, no grouping
CREATE VIEW active_employees AS
SELECT id, name, salary FROM employees WHERE active = TRUE;

UPDATE active_employees SET salary = 90000 WHERE id = 42;
-- Modifies the underlying employees table

-- ❌ Not updatable — has GROUP BY and aggregate
CREATE VIEW dept_totals AS
SELECT department, SUM(salary) AS total FROM employees GROUP BY department;

UPDATE dept_totals SET total = 500000 WHERE department = 'HR';
-- ERROR — which rows should change?
```

- Complex views (aggregates, subqueries, UNION, outer joins) are **read-only**.

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

```ad-tip
**Subquery columns in views are lazy.** If a view defines subqueries in the SELECT list, the engine only executes the subqueries for columns the caller actually references. Unreferenced subquery columns are skipped entirely.

For example, if a view defines `title`, `rating`, and four correlated subquery columns (`category_name`, `num_actors`, `inventory_cnt`, `num_rentals`), but someone runs `SELECT title, rating FROM the_view` — none of the four subqueries execute. If they run `SELECT title, num_actors FROM the_view` — only the `num_actors` subquery runs. This makes it acceptable to put multiple subqueries in a view's SELECT list without worrying that every query pays the cost for all of them.
```

---

### Manual Preaggregation — Views as an Abstraction Layer

- In databases without materialized views (like MySQL), you can achieve the same performance benefit manually by creating a **real table with precomputed results** and redirecting the view to read from it.

**Phase 1: View aggregates live (slow)**

```sql
CREATE VIEW sales_by_film_category AS
SELECT c.name AS category, SUM(p.amount) AS total_sales
FROM category c
JOIN film_category fc ON c.category_id = fc.category_id
JOIN film f ON fc.film_id = f.film_id
JOIN inventory i ON f.film_id = i.film_id
JOIN rental r ON i.inventory_id = r.inventory_id
JOIN payment p ON r.rental_id = p.rental_id
GROUP BY c.name;
-- Every query joins 6 tables and computes SUM on the fly
```

**Phase 2: Create a table with precomputed results**

```sql
CREATE TABLE film_category_sales (
    category    VARCHAR(25),
    total_sales DECIMAL(10,2)
);

INSERT INTO film_category_sales
SELECT c.name, SUM(p.amount)
FROM category c
JOIN film_category fc ON c.category_id = fc.category_id
JOIN ...
GROUP BY c.name;
```

**Phase 3: Redirect the view to the table**

```sql
ALTER VIEW sales_by_film_category AS
SELECT category, total_sales
FROM film_category_sales;
-- Now reads from a flat table — no joins, no aggregation
```

- All existing queries that use `sales_by_film_category` get a performance boost **without any code changes** — the view name is the contract, and the implementation behind it changed transparently.
- The precomputed table must expose the **same columns** (or aliased to match) as the original view so the contract doesn't break.
- Trade-off: the data in the table can become **stale**. You need to refresh it periodically (e.g., a scheduled job that truncates and re-inserts).

```ad-tip
This is essentially a **manual materialized view** pattern — useful in MySQL which doesn't support `CREATE MATERIALIZED VIEW`. The view acts as an interface: consumers depend on the view name, and you can swap the implementation (live query vs precomputed table) without breaking callers. Same idea as an **interface** in C# — swap the implementation, keep the contract.
```

---

### Joining Partitioned Data

- For very large tables, a common design is to **split the table into smaller pieces** by some criteria (usually date ranges) to improve performance. A view then **reunifies** them so consumers don't need to know about the split.

**Without a view — every consumer must know the structure:**

```sql
-- Annoying — and breaks when payment_2026 is added
SELECT * FROM payment_2023
UNION ALL
SELECT * FROM payment_2024
UNION ALL
SELECT * FROM payment_2025;
```

**With a view — consumers query one name:**

```sql
CREATE VIEW payment AS
SELECT * FROM payment_2023
UNION ALL
SELECT * FROM payment_2024
UNION ALL
SELECT * FROM payment_2025;

-- Consumers just write:
SELECT * FROM payment WHERE payment_date > '2024-06-01';
-- They don't know or care that the data is split across three tables
```

- When a new year arrives, add the new table to the view definition — **no consumer code changes**.
- Same abstraction layer pattern as preaggregation above: the view is the contract, the implementation behind it can change freely.

```ad-note
This is different from the database engine's built-in **table partitioning** feature (where a single table is internally partitioned by the engine). Here, the split is done manually with separate tables and a view to stitch them together — a simpler, more portable approach that works on any database.
```
