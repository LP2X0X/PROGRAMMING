---
tags: [sql, performance, indexes]
---

- Indexes come in two main forms based on how they relate to the actual table data.

---

### Clustered Index

- Determines the **physical order** of data on disk. The table data itself is stored sorted by the clustered index key.
- **Only one** clustered index per table (data can only be physically sorted one way).
- In **MySQL/InnoDB**, the primary key IS the clustered index automatically.
- In **SQL Server**, the primary key is clustered by default, but you can choose a different column.

**Analogy**: a phone book sorted by last name. The entries ARE in that order — you don't need a separate lookup.

---

### Non-Clustered Index

- A **separate structure** that stores the indexed column values along with pointers back to the actual rows.
- **Multiple** non-clustered indexes allowed per table.
- Slightly slower than clustered lookups because the database may need a second step (look up the pointer, then fetch the row).

**Analogy**: the index at the back of a textbook. It tells you which page to go to, but the book content isn't sorted by that index.

---

### Covering Index

- A non-clustered index that contains **all columns** needed by a query. The database can satisfy the query entirely from the index without looking up the actual row.
```sql
-- If your query is:
SELECT last_name, department FROM employees WHERE last_name = 'Smith';

-- This covering index satisfies the query completely:
CREATE INDEX idx_cover ON employees(last_name, department);
```

- Covering indexes are very fast because they avoid the "bookmark lookup" to the actual table row.

---

### Composite Index

- An index on **multiple columns**. Column order matters due to the **leftmost prefix rule**:
```sql
CREATE INDEX idx_dept_salary ON employees(department, salary);
```

- This index is used for:
  - `WHERE department = 'Engineering'` — yes (leftmost column)
  - `WHERE department = 'Engineering' AND salary > 50000` — yes (both columns, left to right)
  - `WHERE salary > 50000` — **no** (skips the leftmost column)

```ad-note
Think of a composite index like a phone book sorted by last name, then first name. You can look up by last name, or by last name + first name, but not by first name alone.
```

---

### Summary

| Feature         | Clustered            | Non-Clustered                   |
| --------------- | -------------------- | ------------------------------- |
| Number per table| One                  | Many                            |
| Data storage    | Data sorted by key   | Separate structure with pointers|
| Speed           | Fastest for range scans | Fast, may need row lookup     |
| Created by      | Primary key (usually)| `CREATE INDEX` statement        |
