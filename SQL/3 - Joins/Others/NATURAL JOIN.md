---
tags: [sql, joins]
---

- A `NATURAL JOIN` automatically joins two tables on **all columns that share the same name** — no `ON` clause needed.

```sql
-- Natural join
SELECT * FROM employees NATURAL JOIN departments;

-- Equivalent explicit join (if both tables have a "department_id" column)
SELECT * FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;
```

- The database looks at both tables' column names, finds the ones in common, and joins on all of them using `=`.
- Shared columns appear **once** in the result, not twice.

---

### Why to Avoid It

`NATURAL JOIN` is fragile and widely discouraged:

1. **Silent breakage** — if someone adds a column with the same name to either table (e.g. both get a `name` or `status` column), the join condition silently changes and you get wrong results with no error
2. **Not explicit** — you can't tell what the join condition is just by reading the query. You have to know both tables' schemas
3. **Maintenance risk** — schema changes in one table can break queries that don't even reference that table's new column

```ad-warning
Prefer explicit `INNER JOIN ... ON` over `NATURAL JOIN`. Being explicit about join conditions makes queries self-documenting and immune to schema changes.
```

---

### USING as a Middle Ground

If you want to skip writing the full `ON a.col = b.col` but stay explicit, use `USING`:

```sql
-- USING: explicit about which columns, shorter than ON
SELECT * FROM employees
INNER JOIN departments USING (department_id);

-- Equivalent ON:
SELECT * FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;
```

- `USING` requires the column name to be **identical** in both tables
- Like `NATURAL JOIN`, the shared column appears once in the result
- Unlike `NATURAL JOIN`, you control exactly which columns are used
