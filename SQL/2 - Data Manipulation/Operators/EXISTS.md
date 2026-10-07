---
tags: [sql, dml, operators, subqueries]
---

- `EXISTS` checks whether a subquery returns **any rows at all**. It doesn't care about the values — just: "did at least one row come back?"

```sql
-- Customers who have placed at least one order
SELECT c.name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

- Returns `TRUE` if the subquery produces at least one row, `FALSE` if it produces zero rows.
- `NOT EXISTS` is the inverse — returns TRUE when the subquery produces **no** rows.

---

### How It Works

1. For each row in the outer query, SQL runs the subquery
2. If the subquery returns **any row** → `EXISTS` = TRUE → outer row is included
3. If the subquery returns **nothing** → `EXISTS` = FALSE → outer row is excluded
4. SQL **stops evaluating** as soon as it finds the first matching row (short-circuit)

Because of step 4, `SELECT 1` is convention — the actual values don't matter. `SELECT *`, `SELECT 42`, `SELECT 'hello'` all behave identically. The database never looks at what's selected, only whether a row exists.

---

### EXISTS vs IN

Both can solve the same problems, but they behave differently:

| | `IN` | `EXISTS` |
|---|---|---|
| **Mechanism** | Compares a value against each member of a set using `=` | Checks if the subquery returns any rows |
| **NULL safety** | `NOT IN` breaks when NULLs are in the set (see [[IN, BETWEEN, LIKE]]) | `NOT EXISTS` handles NULLs correctly |
| **Performance** | Builds the full set first, then compares | Short-circuits on first match |
| **Best when** | Small, known set of values | Large tables, checking for existence of related rows |

```sql
-- These are logically equivalent:
WHERE department_id IN (SELECT id FROM departments WHERE location = 'NY')
WHERE EXISTS (SELECT 1 FROM departments d WHERE d.id = e.department_id AND d.location = 'NY')

-- But NOT IN vs NOT EXISTS are NOT equivalent when NULLs are involved:
WHERE id NOT IN (SELECT val FROM t)         -- returns NOTHING if any val is NULL
WHERE NOT EXISTS (SELECT 1 FROM t WHERE t.val = outer.id)  -- works correctly
```

```ad-tip
title: Rule of Thumb
When checking existence of related rows, prefer `EXISTS` / `NOT EXISTS` over `IN` / `NOT IN`. It's NULL-safe and the database can short-circuit the evaluation.
```

---

### Common Patterns

**Find rows WITH related data:**
```sql
-- Customers who have orders
SELECT c.name FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.id
);
```

**Find rows WITHOUT related data:**
```sql
-- Customers who have never ordered
SELECT c.name FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.id
);
```

**Conditional insert (don't insert duplicates):**
```sql
INSERT INTO customers (name, email)
SELECT 'John', 'john@test.com'
WHERE NOT EXISTS (
    SELECT 1 FROM customers WHERE email = 'john@test.com'
);
```

---

### EXISTS Is Always a Correlated Subquery

`EXISTS` almost always references the outer query — that's what makes it useful. A non-correlated `EXISTS` would just be a constant TRUE/FALSE check, which is rarely what you want.

```sql
-- This is always TRUE or always FALSE for every row — pointless
WHERE EXISTS (SELECT 1 FROM departments WHERE location = 'NY')

-- This is useful — correlated to each outer row
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id)
```

See [[Correlated Subqueries]] for more on how correlated subqueries work.
