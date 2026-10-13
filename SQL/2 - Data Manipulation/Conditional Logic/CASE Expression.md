---
tags: [sql, dml, conditional-logic]
---

- `CASE` is SQL's if/else. It returns a value based on conditions — use it anywhere an expression is valid (`SELECT`, `WHERE`, `ORDER BY`, `GROUP BY`, inside aggregate functions).
- The `THEN` clause can return any expression: a string, number, date, `NULL`, a column value, a function call, even another `CASE`. It just has to be a single value — not a row or table.

---

### Simple CASE (Match a Value)

Compares one expression against multiple values — like C#'s `switch`:

```sql
SELECT first_name, last_name,
    CASE rating
        WHEN 'G'     THEN 'General'
        WHEN 'PG'    THEN 'Parental Guidance'
        WHEN 'PG-13' THEN 'Parents Cautioned'
        WHEN 'R'     THEN 'Restricted'
        ELSE 'Unknown'
    END AS rating_label
FROM film;
```

---

### Searched CASE (Evaluate Conditions)

Each `WHEN` has its own boolean condition — like if/else if:

```sql
SELECT first_name, last_name, salary,
    CASE
        WHEN salary >= 100000 THEN 'High'
        WHEN salary >= 50000  THEN 'Mid'
        ELSE 'Low'
    END AS salary_tier
FROM employees;
```

- Conditions are evaluated **top to bottom** — the first `WHEN` that is TRUE wins
- `ELSE` is optional but recommended — without it, unmatched rows return `NULL`

---

### CASE in Different Clauses

**In ORDER BY** — custom sort order:
```sql
SELECT name, status
FROM orders
ORDER BY
    CASE status
        WHEN 'urgent'    THEN 1
        WHEN 'normal'    THEN 2
        WHEN 'low'       THEN 3
        ELSE 4
    END;
```

**In WHERE** — conditional filtering:
```sql
SELECT * FROM employees
WHERE salary >
    CASE department
        WHEN 'Engineering' THEN 80000
        WHEN 'Sales'       THEN 50000
        ELSE 60000
    END;
```

**Inside aggregate functions** — conditional counting/summing:
```sql
SELECT
    SUM(CASE WHEN rating = 'PG' THEN 1 ELSE 0 END) AS pg_count,
    SUM(CASE WHEN rating = 'R'  THEN 1 ELSE 0 END) AS r_count
FROM film;
```

```ad-tip
title: CASE Inside Aggregates
This pattern is the SQL equivalent of "pivot" — turning row values into columns. Very common in reporting queries.
```

---

### Simple vs Searched

| Simple CASE | Searched CASE |
|---|---|
| `CASE x WHEN val1 THEN ...` | `CASE WHEN condition THEN ...` |
| Compares one value against many | Each `WHEN` has its own condition |
| Equality only (`=`) | Any comparison (`>`, `<`, `BETWEEN`, `LIKE`, etc.) |
| Like C# `switch` | Like C# `if/else if` |

```ad-note
`CASE` is an **expression**, not a statement. It evaluates to a single value — just like C#'s ternary `? :`. That's why you can put it **inside** functions, not just in `SELECT`:

​```sql
-- CASE inside SUM — works because CASE returns a value (1 or 0)
SUM(CASE WHEN rating = 'PG' THEN 1 ELSE 0 END)
​```

Wherever SQL expects a value, you can put a `CASE` there. It doesn't control flow like `IF` in stored procedures. See [[Stored Procedures]] for procedural `IF/ELSE`.
```
