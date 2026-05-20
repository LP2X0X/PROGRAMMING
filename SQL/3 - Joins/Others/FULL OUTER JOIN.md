---
tags: [sql, joins]
---

- A `FULL OUTER JOIN` returns **all rows from both tables**. Where there's no match on either side, the missing columns are filled with `NULL`.
- It combines the behavior of [[LEFT and RIGHT JOIN]] — you get unmatched rows from both sides.

---

### Syntax

```sql
SELECT e.name, d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.id;
```

| name    | department_name |
| ------- | --------------- |
| Alice   | Engineering     |
| Bob     | Marketing       |
| Charlie | NULL            |
| NULL    | Sales           |

- Charlie has no department. Sales has no employees. Both appear.

---

### Use Case: Finding Mismatches

```sql
-- Find all unmatched records between two datasets
SELECT a.id AS source_a, b.id AS source_b
FROM dataset_a a
FULL OUTER JOIN dataset_b b ON a.id = b.id
WHERE a.id IS NULL OR b.id IS NULL;
```

---

### MySQL Workaround

```ad-note
MySQL does **not** support `FULL OUTER JOIN`. Emulate it with a `UNION` of `LEFT JOIN` and `RIGHT JOIN`:
```

```sql
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id

UNION

SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

- Use `UNION ALL` instead of `UNION` if you want to keep duplicates (faster, but may include rows that matched in both queries).
