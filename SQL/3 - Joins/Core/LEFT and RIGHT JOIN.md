---
tags: [sql, joins]
---

- A `LEFT JOIN` (or `LEFT OUTER JOIN`) returns **all rows from the left table**, plus matching rows from the right table. Where there's no match, the right side columns are filled with `NULL`.
- A `RIGHT JOIN` is the mirror — all rows from the right table, NULLs for unmatched left rows.

---

### LEFT JOIN

```sql
SELECT c.name, o.order_id, o.total
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id;
```

| name    | order_id | total  |
| ------- | -------- | ------ |
| Alice   | 101      | 250.00 |
| Alice   | 102      | 75.00  |
| Bob     | 103      | 500.00 |
| Charlie | NULL     | NULL   |

- Charlie has no orders, but still appears in the result with NULLs.

---

### Finding Unmatched Rows

- One of the most useful patterns — find rows in one table with **no corresponding row** in another:
```sql
-- Customers who have never placed an order
SELECT c.name
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.id IS NULL;
```

```ad-tip
This pattern (LEFT JOIN + WHERE right.key IS NULL) is often faster than `NOT IN` or `NOT EXISTS` and avoids the NULL pitfall of `NOT IN`. See [[IS NULL]].
```

---

### RIGHT JOIN

```sql
SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

- Returns all departments, even those with no employees.
- In practice, `RIGHT JOIN` is rarely used — you can always rewrite it as a `LEFT JOIN` by swapping the table order. Most codebases standardize on `LEFT JOIN` for consistency.

---

### LEFT JOIN vs INNER JOIN

- `INNER JOIN`: only rows with matches on both sides.
- `LEFT JOIN`: all left rows + matches. NULLs for no match.
- Use `INNER JOIN` when you only want complete data. Use `LEFT JOIN` when you need to keep all rows from one side regardless of matches.

```ad-warning
Adding a `WHERE` condition on the right table of a `LEFT JOIN` can accidentally convert it into an `INNER JOIN`:
```

```sql
-- This filters out NULLs, effectively making it an INNER JOIN:
SELECT c.name, o.order_id
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.status = 'shipped';  -- NULLs are excluded!

-- To keep the LEFT JOIN behavior, move the filter into ON:
SELECT c.name, o.order_id
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id AND o.status = 'shipped';
```
