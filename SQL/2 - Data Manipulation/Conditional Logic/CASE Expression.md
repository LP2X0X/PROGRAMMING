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
This pattern is the SQL equivalent of "pivot" — turning row values into columns. Very common in reporting queries. See the CASE + MAX Pivot Trick below for the full technique.
```

---

### CASE + MAX Pivot Trick

A technique to turn **rows into columns** when your database doesn't have a native `PIVOT` operator (MySQL doesn't).

**Source data (rows):**

```
student | subject | score
--------|---------|------
Park    | Math    | 90
Park    | English | 85
Park    | Science | 92
Kim     | Math    | 78
Kim     | English | 91
Kim     | Science | 80
```

**Desired output (columns):**

```
student | Math | English | Science
--------|------|---------|--------
Park    | 90   | 85      | 92
Kim     | 78   | 91      | 80
```

#### Step 1: CASE Alone — Spreads Values but Doesn't Collapse

```sql
SELECT
    student,
    CASE WHEN subject = 'Math'    THEN score ELSE NULL END AS Math,
    CASE WHEN subject = 'English' THEN score ELSE NULL END AS English,
    CASE WHEN subject = 'Science' THEN score ELSE NULL END AS Science
FROM scores;
```

Each row only matches one CASE — the rest are NULL:

```
student | Math | English | Science
--------|------|---------|--------
Park    | 90   | NULL    | NULL      -- row 1: only Math matched
Park    | NULL | 85      | NULL      -- row 2: only English matched
Park    | NULL | NULL    | 92        -- row 3: only Science matched
Kim     | 78   | NULL    | NULL
Kim     | NULL | 91      | NULL
Kim     | NULL | NULL    | 80
```

Still multiple rows per student — not collapsed yet.

#### Step 2: MAX + GROUP BY — Collapses into One Row

```sql
SELECT
    student,
    MAX(CASE WHEN subject = 'Math'    THEN score ELSE NULL END) AS Math,
    MAX(CASE WHEN subject = 'English' THEN score ELSE NULL END) AS English,
    MAX(CASE WHEN subject = 'Science' THEN score ELSE NULL END) AS Science
FROM scores
GROUP BY student;
```

`GROUP BY student` groups all rows for each student. `MAX` picks the single non-NULL value from each column (aggregates skip NULLs):

```
MAX(90, NULL, NULL) = 90
MAX(NULL, 85, NULL) = 85
MAX(NULL, NULL, 92) = 92
```

Result:

```
student | Math | English | Science
--------|------|---------|--------
Park    | 90   | 85      | 92
Kim     | 78   | 91      | 80
```

```ad-note
`MAX` is the convention, but `MIN` or `SUM` also work — they all ignore NULLs and return the single non-NULL value. `MAX` is just the most common choice.
```

#### Limitation

You must **hardcode** each CASE expression — one per value you want to pivot. If new subjects are added, you must update the query. For a truly dynamic number of columns, combine this with [[Dynamic SQL]] to generate the CASE expressions at runtime.

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
