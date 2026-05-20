---
tags: [sql, dml, operators]
---

- `NULL` in SQL is **not a value** — it represents the **absence of a value**. It means "unknown" or "not applicable."

---

### Checking for NULL

```sql
-- Correct:
SELECT * FROM employees WHERE manager_id IS NULL;
SELECT * FROM employees WHERE manager_id IS NOT NULL;
```

```ad-warning
**Never use `=` or `<>` to compare with NULL.** Any comparison with `NULL` returns `NULL` (unknown), not `TRUE` or `FALSE`:
```

```sql
-- WRONG — this returns NO rows, even if manager_id IS null:
SELECT * FROM employees WHERE manager_id = NULL;

-- WRONG — this also returns NO rows:
SELECT * FROM employees WHERE manager_id <> NULL;
```

---

### Three-Valued Logic

- SQL uses **three-valued logic**: `TRUE`, `FALSE`, and `NULL` (unknown).
- Any arithmetic or comparison involving `NULL` produces `NULL`:
```sql
SELECT 5 + NULL;    -- NULL
SELECT 5 > NULL;    -- NULL
SELECT NULL = NULL;  -- NULL (not TRUE!)
```

| Expression          | Result   |
| ------------------- | -------- |
| `TRUE AND NULL`     | NULL     |
| `FALSE AND NULL`    | FALSE    |
| `TRUE OR NULL`      | TRUE     |
| `FALSE OR NULL`     | NULL     |
| `NOT NULL`          | NULL     |

---

### Handling NULLs in Practice

- Use [[COALESCE and IFNULL]] to provide default values:
```sql
SELECT 
    first_name,
    COALESCE(phone, 'N/A') AS phone
FROM employees;
```

- NULLs in [[Aggregate Functions]]:
  - `COUNT(*)` counts all rows including NULLs.
  - `COUNT(column)` counts only non-NULL values.
  - `SUM`, `AVG`, `MIN`, `MAX` all **ignore** NULL values.

- NULLs in [[ORDER BY]]: NULLs sort first or last depending on the RDBMS. Use `NULLS FIRST` or `NULLS LAST` (PostgreSQL, Oracle) to control this.
