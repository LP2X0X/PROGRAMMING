---
tags: [sql, dml, operators]
---

- These operators provide cleaner ways to express common filtering patterns in [[WHERE]] clauses.

---

### IN

- Checks if a value matches **any value in a list**:
```sql
SELECT * FROM employees
WHERE department IN ('Engineering', 'Marketing', 'Sales');

-- Equivalent to:
-- WHERE department = 'Engineering' 
--    OR department = 'Marketing' 
--    OR department = 'Sales'
```

- `IN` also works with subqueries:
```sql
SELECT * FROM employees
WHERE department_id IN (
    SELECT id FROM departments WHERE location = 'New York'
);
```

- `NOT IN`: matches values NOT in the list.

```ad-warning
`NOT IN` behaves unexpectedly with `NULL` values. If the subquery returns any `NULL`, `NOT IN` returns no rows at all. Use `NOT EXISTS` instead when NULLs are possible. See [[IS NULL]].
```

---

### BETWEEN

- Checks if a value is within a **range (inclusive on both ends)**:
```sql
SELECT * FROM products
WHERE price BETWEEN 10 AND 50;

-- Equivalent to:
-- WHERE price >= 10 AND price <= 50
```

- Works with dates too:
```sql
SELECT * FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31';
```

- `NOT BETWEEN`: values outside the range.

---

### LIKE

- Pattern matching for strings using wildcards:
  - `%` — matches **zero or more** characters.
  - `_` — matches **exactly one** character.

```sql
-- Names starting with 'J'
SELECT * FROM employees WHERE first_name LIKE 'J%';

-- Names ending with 'son'
SELECT * FROM employees WHERE last_name LIKE '%son';

-- Names containing 'an' anywhere
SELECT * FROM employees WHERE first_name LIKE '%an%';

-- Exactly 4-letter names
SELECT * FROM employees WHERE first_name LIKE '____';
```

- `NOT LIKE`: rows that don't match the pattern.
- Case sensitivity depends on the RDBMS and collation:
  - MySQL: case-insensitive by default.
  - PostgreSQL: case-sensitive. Use `ILIKE` for case-insensitive matching.

```ad-tip
`LIKE` with a leading `%` (e.g., `LIKE '%son'`) cannot use an index and causes a full table scan. For better performance, prefer leading patterns (`LIKE 'J%'`) or use full-text search for complex text matching. See [[What are Indexes]].
```
