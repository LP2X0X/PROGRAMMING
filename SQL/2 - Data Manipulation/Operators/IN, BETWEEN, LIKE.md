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
title: NOT IN + NULL = Silent Empty Result
`NOT IN` behaves unexpectedly with `NULL` values. If the set contains any `NULL`, `NOT IN` returns **no rows at all**.

**Why:** `IN` and `NOT IN` work by comparing the left side to each member using `=`. But `= NULL` always evaluates to **UNKNOWN** (not FALSE), because NULL means "I don't know this value."

​```sql
-- Set: (1, 3, NULL)
WHERE 2 NOT IN (1, 3, NULL)

-- SQL evaluates:
-- 2 = 1    → FALSE
-- 2 = 3    → FALSE
-- 2 = NULL → UNKNOWN
​```

`NOT IN` needs **every** comparison to be FALSE to return TRUE. One UNKNOWN poisons the whole result → the row is excluded.

`IN` is less dangerous — it only needs **one** TRUE to include the row, so the UNKNOWN from NULL is harmless as long as another value matches. But `NOT IN` requires certainty about every member, and NULL makes that impossible.

**Fix:** filter out NULLs or use `NOT EXISTS`:
​```sql
-- Option 1: exclude NULLs
WHERE id NOT IN (SELECT val FROM t WHERE val IS NOT NULL)

-- Option 2: use NOT EXISTS (NULL-safe)
WHERE NOT EXISTS (SELECT 1 FROM t WHERE t.val = outer.id)
​```
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

```ad-warning
BETWEEN is inclusive. Which mean both is lower an upper range limits will be included in the filter itself.  
```

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
