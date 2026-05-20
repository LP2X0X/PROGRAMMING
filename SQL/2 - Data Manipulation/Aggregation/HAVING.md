---
tags: [sql, dml, aggregation]
---

- `HAVING` filters **groups** after aggregation. It is to `GROUP BY` what `WHERE` is to `FROM`.

---

### HAVING vs WHERE

| Clause    | Filters          | When it runs        | Can use aggregates? |
| --------- | ---------------- | ------------------- | ------------------- |
| `WHERE`   | Individual rows  | Before `GROUP BY`   | No                  |
| `HAVING`  | Groups           | After `GROUP BY`    | Yes                 |

```sql
-- WHERE filters rows BEFORE grouping:
SELECT department, AVG(salary) AS avg_salary
FROM employees
WHERE salary > 40000           -- exclude low-salary rows first
GROUP BY department;

-- HAVING filters groups AFTER grouping:
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 70000;    -- only departments with high average
```

---

### Common Pattern: Finding Duplicates

```sql
SELECT email, COUNT(*) AS count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
-- Returns emails that appear more than once
```

---

### HAVING with Multiple Conditions

```sql
SELECT department, COUNT(*) AS cnt, AVG(salary) AS avg_sal
FROM employees
GROUP BY department
HAVING COUNT(*) >= 5 AND AVG(salary) > 60000;
```

---

### HAVING Without GROUP BY

- When used without `GROUP BY`, `HAVING` treats the entire result as one group:
```sql
SELECT AVG(salary) AS avg_salary
FROM employees
HAVING AVG(salary) > 50000;
-- Returns the average only if it's above 50000, otherwise returns no rows
```

```ad-tip
If your filter doesn't involve an aggregate function, use `WHERE` instead of `HAVING` — it's more efficient because it reduces rows before the aggregation step.
```
