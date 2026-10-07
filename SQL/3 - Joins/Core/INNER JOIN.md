---
tags: [sql, joins]
---

- An `INNER JOIN` returns only the rows where there is a **match in both tables**. Unmatched rows from either side are excluded.
- `JOIN` without a prefix is shorthand for `INNER JOIN`.

---

### Syntax

```sql
SELECT e.first_name, e.last_name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
```

- **Table aliases** (`e`, `d`) keep the query readable — especially important when joining multiple tables.

---

### How Joins Actually Work

At its core, every join is a [[CROSS JOIN|Cartesian product]] with a filter:

```
for each row in table_A:
    for each row in table_B:
        if ON condition is TRUE:
            include this pair in result
```

The database takes every possible combination of rows from both tables, and keeps only the pairs where `ON` returns TRUE. The `ON` clause is just a boolean expression — it doesn't know or care what kind of comparison you use:

```sql
ON a.id = b.id                          -- equality (most common)
ON a.total BETWEEN b.low AND b.high     -- range
ON a.name LIKE b.pattern                -- pattern
ON a.score > b.threshold                -- inequality
```

All valid. We usually join on `=` because the most common relationship is "this ID matches that ID" — but equality is just the most frequent condition, not the only option.

---

### Example

Given:

| employees        |            | departments |                 |
| ---------------- | ---------- | ----------- | --------------- |
| name             | dept_id    | id          | dept_name       |
| Alice            | 1          | 1           | Engineering     |
| Bob              | 2          | 2           | Marketing       |
| Charlie          | 99         | 3           | Sales           |

```sql
SELECT e.name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.id;
```

Result: Alice → Engineering, Bob → Marketing. Charlie is excluded (dept_id 99 has no match). Sales is excluded (no employee references it).

---

### Joining on Multiple Conditions

```sql
SELECT *
FROM order_items oi
INNER JOIN products p 
    ON oi.product_id = p.id 
    AND oi.warehouse_id = p.warehouse_id;
```

---

### Joining Multiple Tables

```sql
SELECT o.order_id, c.name, p.product_name
FROM orders o
INNER JOIN customers c ON o.customer_id = c.id
INNER JOIN order_items oi ON o.id = oi.order_id
INNER JOIN products p ON oi.product_id = p.id;
```

```ad-note
The join condition goes in the `ON` clause, not `WHERE`. While putting join conditions in `WHERE` works for inner joins, it fails for outer joins and is less readable. Keep join logic in `ON` and filter logic in `WHERE`. See also [[LEFT and RIGHT JOIN]], [[Self Join]].
```
