---
tags: [sql, performance, indexes]
---

- Indexing is about balance — too few indexes and queries are slow; too many and writes suffer.

---

### DO Index These Columns

- Columns frequently used in **WHERE** clauses:
```sql
-- If you often query by department:
CREATE INDEX idx_department ON employees(department);
```

- Columns used in **JOIN ON** conditions:
```sql
-- Foreign keys should almost always be indexed:
CREATE INDEX idx_order_customer ON orders(customer_id);
```

- Columns used in **ORDER BY** or **GROUP BY**:
```sql
CREATE INDEX idx_created ON orders(created_at);
```

- Columns with **high cardinality** (many distinct values) — e.g., email, user_id, timestamp.

---

### DON'T Index These Columns

- **Small tables** (under a few thousand rows) — full scan is fast enough.
- **Low-cardinality columns** — e.g., boolean, gender, status with only 2-3 values. The index doesn't narrow results enough to be useful.
- **Columns rarely used in queries** — unused indexes just waste space and slow writes.
- **Frequently updated columns** — every update also updates the index.

---

### Composite Index Tips

- Put the **most selective column first** (the one that narrows results the most):
```sql
-- Good: status has few values, created_at is selective
CREATE INDEX idx_orders ON orders(created_at, status);

-- Less useful: filtering by status alone is too broad
CREATE INDEX idx_orders ON orders(status, created_at);
```

- **Leftmost prefix rule**: a composite index on `(A, B, C)` can be used for queries filtering on `A`, `A+B`, or `A+B+C`, but not `B` or `C` alone. See [[Clustered vs Non-Clustered Indexes]].

---

### Index Maintenance

```sql
-- Create:
CREATE INDEX idx_name ON table(column);

-- Drop when no longer needed:
DROP INDEX idx_name ON table;

-- View existing indexes (MySQL):
SHOW INDEX FROM table_name;

-- View existing indexes (PostgreSQL):
SELECT * FROM pg_indexes WHERE tablename = 'table_name';
```

```ad-tip
As a rule of thumb: index every [[Foreign Key]] column, index columns you frequently filter or sort by, and then use [[EXPLAIN and Query Plans]] to verify performance. Don't guess — measure.
```
