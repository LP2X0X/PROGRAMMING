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

### How It Works

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
