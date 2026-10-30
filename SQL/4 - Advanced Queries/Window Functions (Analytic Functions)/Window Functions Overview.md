---
tags: [sql, window-functions, advanced]
---

- **Window functions** perform calculations across a set of rows related to the current row — without collapsing rows like [[GROUP BY]] does. Every row in the result keeps its individual values.

---

### Key Difference from GROUP BY

```sql
-- GROUP BY: collapses rows (5 departments → 5 rows)
SELECT department, AVG(salary) FROM employees GROUP BY department;

-- Window function: preserves all rows, adds the aggregate alongside
SELECT name, department, salary,
       AVG(salary) OVER (PARTITION BY department) AS dept_avg
FROM employees;
-- Every employee row is kept, each showing their department's average
```

---

### Syntax

```sql
function_name() OVER (
    PARTITION BY column    -- optional: divide rows into groups
    ORDER BY column        -- optional: define order within each partition
    frame_clause           -- optional: define which rows to include
)
```

- **PARTITION BY**: divides rows into groups (like GROUP BY, but without collapsing). If omitted, the entire [[Result Set|result set]] is one partition.
- **ORDER BY**: defines the order of rows within each partition. Required for ranking and cumulative functions.
- **Frame clause**: defines the "window" of rows relative to the current row (e.g., `ROWS BETWEEN 3 PRECEDING AND CURRENT ROW`).

---

### Types of Window Functions

| Category        | Functions                                     | Notes                                  |
| --------------- | --------------------------------------------- | -------------------------------------- |
| Ranking         | `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`   | See [[ROW_NUMBER, RANK, DENSE_RANK]]   |
| Offset          | `LAG`, `LEAD`, `FIRST_VALUE`, `LAST_VALUE`     | See [[LAG and LEAD]]                   |
| Aggregate       | `SUM`, `AVG`, `COUNT`, `MIN`, `MAX`            | See [[Aggregate Window Functions]]     |

---

### Window Frame

- The frame defines which rows relative to the current row are included in the calculation:
```sql
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW     -- current row + 2 before
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW  -- all rows from start to current
ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING      -- current row + 1 before + 1 after
```

- Default frame (when ORDER BY is present): `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`.

---

### Availability

```ad-note
Window functions are supported in MySQL 8.0+, PostgreSQL 8.4+, SQL Server 2012+, SQLite 3.25+, MariaDB 10.2+, and Oracle. If you're on an older MySQL version, you'll need to use subqueries or variables instead.
```
