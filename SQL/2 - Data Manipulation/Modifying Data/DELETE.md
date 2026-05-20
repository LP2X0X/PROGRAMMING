---
tags: [sql, dml, modifying-data]
---

- `DELETE` removes rows from a table.

---

### Basic Syntax

```sql
DELETE FROM employees
WHERE employee_id = 42;
```

---

### DELETE Without WHERE

```ad-warning
A `DELETE` without `WHERE` removes **all rows** from the table. The table structure remains, but all data is gone.
```

```sql
-- DANGER: deletes every row in the table
DELETE FROM employees;
```

---

### DELETE with JOIN

```sql
-- MySQL: delete orders for inactive customers
DELETE o
FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE c.active = FALSE;
```

```sql
-- PostgreSQL
DELETE FROM orders
USING customers
WHERE orders.customer_id = customers.id
  AND customers.active = FALSE;
```

---

### DELETE vs TRUNCATE vs DROP

| Operation    | Removes     | Logged?          | Rollback?            | Resets Auto-Increment? |
| ------------ | ----------- | ---------------- | -------------------- | ---------------------- |
| `DELETE`     | Specific rows (or all with no `WHERE`) | Row-by-row | Yes | No |
| `TRUNCATE`   | All rows    | Minimal logging  | Depends on RDBMS     | Yes                    |
| `DROP TABLE` | Entire table (structure + data) | DDL log | Depends on RDBMS | N/A |

- Use `DELETE` when you need a `WHERE` clause or transaction safety.
- Use `TRUNCATE` when you want to quickly empty a table and don't need row-level control.
- Use `DROP` when you want to remove the table entirely. See [[DROP and TRUNCATE]] for more.

---

### Soft Delete Pattern

- Instead of actually deleting rows, mark them as deleted:
```sql
UPDATE employees
SET deleted_at = NOW()
WHERE employee_id = 42;
```
- Queries then filter with `WHERE deleted_at IS NULL` to see only active records.
- Benefits: data recovery is possible, audit trail is preserved.
- Drawback: every query must remember to exclude soft-deleted rows.
