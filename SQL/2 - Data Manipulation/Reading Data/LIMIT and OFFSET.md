---
tags: [sql, dml, reading-data]
---

- `LIMIT` restricts the number of rows returned. Essential for pagination and performance.

---

### Syntax by RDBMS

```sql
-- MySQL, PostgreSQL, MariaDB, SQLite:
SELECT * FROM products ORDER BY price DESC LIMIT 10;

-- SQL Server:
SELECT TOP 10 * FROM products ORDER BY price DESC;

-- Standard SQL (ANSI):
SELECT * FROM products ORDER BY price DESC
FETCH FIRST 10 ROWS ONLY;
```

---

### OFFSET for Pagination

- `OFFSET` skips a number of rows before returning results:
```sql
-- Page 1: rows 1-10
SELECT * FROM products ORDER BY id LIMIT 10 OFFSET 0;

-- Page 2: rows 11-20
SELECT * FROM products ORDER BY id LIMIT 10 OFFSET 10;

-- Page 3: rows 21-30
SELECT * FROM products ORDER BY id LIMIT 10 OFFSET 20;
```

- MySQL shorthand: `LIMIT offset, count`:
```sql
SELECT * FROM products ORDER BY id LIMIT 10, 10;  -- skip 10, take 10
```

```ad-warning
**Large OFFSETs are slow.** `OFFSET 10000` means the database reads and discards 10,000 rows before returning results. For large datasets, use **keyset pagination** (also called "seek method") instead:
```

```sql
-- Instead of: LIMIT 10 OFFSET 10000
-- Use the last seen ID:
SELECT * FROM products
WHERE id > 10000       -- last id from previous page
ORDER BY id
LIMIT 10;
```

- Keyset pagination is O(1) regardless of page number, while OFFSET is O(n).

---

### Common Pattern: Top N

```sql
-- Top 5 highest-paid employees
SELECT first_name, last_name, salary
FROM employees
ORDER BY salary DESC
LIMIT 5;
```

```ad-tip
Always pair `LIMIT` with `ORDER BY`. Without `ORDER BY`, the "top N" rows are arbitrary — the database doesn't guarantee any particular order. See [[ORDER BY]].
```
