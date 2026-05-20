---
tags: [sql, ddl, table-management]
---

- Both `DROP` and `TRUNCATE` remove data, but they differ in scope and behavior.

---

### DROP TABLE

- Removes the table **entirely** — structure, data, indexes, constraints, all gone:
```sql
DROP TABLE employees;
```

- Use `IF EXISTS` to avoid errors:
```sql
DROP TABLE IF EXISTS temp_results;
```

- `DROP DATABASE` removes an entire database:
```sql
DROP DATABASE IF EXISTS test_db;
```

```ad-warning
`DROP` is **irreversible** in most RDBMS. There is no undo. Always double-check before dropping, especially in production.
```

---

### TRUNCATE TABLE

- Removes **all rows** from a table but keeps the table structure (columns, constraints, indexes):
```sql
TRUNCATE TABLE orders;
```

---

### TRUNCATE vs DELETE

| Feature              | `DELETE` (no WHERE)      | `TRUNCATE`               |
| -------------------- | ------------------------ | ------------------------ |
| Removes              | All rows                 | All rows                 |
| Table structure      | Kept                     | Kept                     |
| WHERE clause         | Supported                | Not supported            |
| Logging              | Row-by-row (slow)        | Minimal/page-level (fast)|
| Triggers fire        | Yes                      | No                       |
| Resets auto-increment| No                       | Yes                      |
| Rollback             | Yes                      | Depends on RDBMS         |
| Foreign key check    | Yes                      | Cannot truncate if referenced |

- Use `TRUNCATE` when you want to quickly empty a table and reset auto-increment counters.
- Use [[DELETE]] when you need a `WHERE` clause, trigger execution, or transaction rollback safety.

```ad-tip
`TRUNCATE` can be 10–100x faster than `DELETE` on large tables because it doesn't generate individual row-level log entries. Use it for clearing staging tables, test data, or temporary results.
```
