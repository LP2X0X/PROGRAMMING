---
tags: [sql, fundamentals]
---

- A **column alias** acts like adding a new column to the [[Result Set|result set]], but it doesn't create anything in the actual table. It only exists for that query's output.

---

### Syntax

```sql
SELECT first_name, last_name,
       salary * 12 AS annual_salary
FROM employees;
```

- `AS` is optional — `salary * 12 annual_salary` works too, but `AS` is clearer.

---

### Execution Order Matters

- `WHERE` is evaluated **before** `SELECT`, so you **cannot** filter on a column alias in `WHERE`. See [[SQL Syntax Basics]].
- `ORDER BY` is evaluated **after** `SELECT`, so it **can** reference column aliases. See [[ORDER BY]].
- To filter on an alias, use [[HAVING]] (for aggregations) or wrap in a subquery.
