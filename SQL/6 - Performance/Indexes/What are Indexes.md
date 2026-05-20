---
tags: [sql, performance, indexes]
---

- An **index** is a data structure (usually a B-tree) that speeds up data retrieval — like the index at the back of a book that lets you jump to the right page instead of reading cover-to-cover.

---

### Without an Index

- The database performs a **full table scan** — it reads every row in the table to find matches.
- For a table with 1 million rows, this means examining all 1 million rows even if only 5 match.

### With an Index

- The database uses the index to jump directly to the relevant rows.
- Lookup time goes from O(n) to approximately O(log n) for B-tree indexes.

---

### Trade-offs

| Benefit                       | Cost                                    |
| ----------------------------- | --------------------------------------- |
| Much faster `SELECT` queries  | Slower `INSERT`, `UPDATE`, `DELETE`     |
| Faster `JOIN` and `ORDER BY`  | Additional disk space                   |
| Faster `WHERE` filtering      | Index maintenance overhead              |

- Every time data is modified, the database must also update all affected indexes. More indexes = slower writes.

---

### Automatically Created Indexes

- The database **automatically** creates an index on:
  - **PRIMARY KEY** columns — see [[Primary Key]]
  - **UNIQUE** constraint columns — see [[Unique, Not Null, Check, Default]]
- You don't need to manually create indexes for these.

---

### Creating and Dropping Indexes

```sql
-- Create an index:
CREATE INDEX idx_employee_last_name ON employees(last_name);

-- Create a unique index:
CREATE UNIQUE INDEX idx_user_email ON users(email);

-- Drop an index:
DROP INDEX idx_employee_last_name ON employees;  -- MySQL
DROP INDEX idx_employee_last_name;               -- PostgreSQL
```

```ad-tip
See [[When to Use Indexes]] for guidelines on which columns to index, and [[EXPLAIN and Query Plans]] for how to verify that your indexes are actually being used.
```
