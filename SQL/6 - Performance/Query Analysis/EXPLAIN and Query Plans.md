---
tags: [sql, performance, query-analysis]
---

- `EXPLAIN` shows **how** the database will execute a query — which indexes it uses, how many rows it scans, and in what order it processes tables.

---

### Basic Usage

```sql
-- MySQL:
EXPLAIN SELECT * FROM employees WHERE department = 'Engineering';

-- PostgreSQL (with actual timing):
EXPLAIN ANALYZE SELECT * FROM employees WHERE department = 'Engineering';

-- SQL Server:
SET STATISTICS IO ON;
-- or use the graphical execution plan in SSMS
```

---

### MySQL EXPLAIN Key Columns

| Column         | What it tells you                                      |
| -------------- | ------------------------------------------------------ |
| `type`         | How the table is accessed (best → worst below)         |
| `possible_keys`| Which indexes the optimizer considered                 |
| `key`          | Which index was actually chosen                        |
| `rows`         | Estimated number of rows to examine                    |
| `Extra`        | Additional info (e.g., "Using index", "Using filesort")|

### Access Types (best to worst)

| Type     | Meaning                                                |
| -------- | ------------------------------------------------------ |
| `const`  | Single row via primary key or unique index (fastest)   |
| `eq_ref` | One row per join via unique index                      |
| `ref`    | Multiple rows via non-unique index                     |
| `range`  | Index range scan (BETWEEN, <, >, IN)                   |
| `index`  | Full index scan (reads entire index, not table)        |
| `ALL`    | **Full table scan** (slowest — usually means missing index) |

---

### Common Performance Problems

1. **Full table scan (`ALL`)** — add an index on the filtered column.
2. **Using filesort** — the sort can't use an index. Consider adding an index that matches the ORDER BY.
3. **Using temporary** — a temp table is created (often for GROUP BY or DISTINCT). May indicate need for optimization.
4. **SELECT \*** — fetches all columns even if you only need a few. Prevents covering index usage.

---

### Index-Breaking Patterns

- **Functions on indexed columns** disable the index:
```sql
-- BAD: index on created_at won't be used
WHERE YEAR(created_at) = 2024

-- GOOD: rewrite as a range
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01'
```

- **Implicit type conversion** can prevent index usage:
```sql
-- BAD: if phone is VARCHAR, comparing to INT causes conversion
WHERE phone = 5551234

-- GOOD: match the column type
WHERE phone = '5551234'
```

- **Leading wildcards** in LIKE:
```sql
WHERE name LIKE '%smith'  -- full scan (no index)
WHERE name LIKE 'smith%'  -- index scan (good)
```

```ad-tip
Workflow: write your query → run `EXPLAIN` → check for `ALL` type or large `rows` estimate → add or adjust indexes → run `EXPLAIN` again to verify improvement. See [[What are Indexes]] and [[When to Use Indexes]].
```
