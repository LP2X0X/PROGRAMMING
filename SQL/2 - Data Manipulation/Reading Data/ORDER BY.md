---
tags: [sql, dml, reading-data]
---

- `ORDER BY` sorts the result set. Without it, the order of rows is **not guaranteed** — the database can return rows in any order.

---

### Basic Syntax

```sql
SELECT * FROM employees
ORDER BY last_name ASC;  -- ascending (default)

SELECT * FROM employees
ORDER BY salary DESC;    -- descending
```

- `ASC` is the default and can be omitted.

---

### Sorting by Multiple Columns

- Rows are sorted by the first column; ties are broken by the second column, and so on:
```sql
SELECT * FROM employees
ORDER BY department ASC, salary DESC;
-- Sort by department A-Z, then within each department by salary highest first
```

---

### Sorting by Column Position

```sql
SELECT first_name, last_name, salary
FROM employees
ORDER BY 3 DESC;  -- sorts by the 3rd column (salary)
```

- This works but is **fragile** — if you reorder the `SELECT` columns, the sort changes. Prefer using column names.

---

### Sorting by Alias or Expression

```sql
SELECT first_name, salary * 12 AS annual_salary
FROM employees
ORDER BY annual_salary DESC;

-- Or sort by an expression directly:
ORDER BY salary * 12 DESC;
```

- `ORDER BY` is evaluated **after** `SELECT`, so it can reference column aliases. See [[SQL Syntax Basics]] for execution order.

---

### NULL Ordering

- Default NULL position varies by RDBMS:
  - MySQL, SQL Server: NULLs sort as the **lowest** value.
  - PostgreSQL, Oracle: NULLs sort as the **highest** value.

- PostgreSQL and Oracle support explicit control:
```sql
ORDER BY column ASC NULLS LAST;
ORDER BY column DESC NULLS FIRST;
```

- In MySQL, use a workaround:
```sql
ORDER BY column IS NULL, column ASC;
-- IS NULL returns 0 (false) for non-null, 1 (true) for null — pushes NULLs to the end
```
