---
tags: [database, fundamentals, null]
---

**Prerequisite:** [[05 - Foreign Keys and Relationships]]

- `NULL` is one of the most important — and most misunderstood — concepts in SQL. Getting it wrong causes subtle, hard-to-find bugs in queries, reports, and application logic.
- `NULL` is **not** zero. It is **not** an empty string. It is **not** false. `NULL` means **"unknown"** or **"no value."**

---

### What NULL Represents

- `NULL` represents the **absence of data**. It means the value is unknown, not applicable, or has not been provided.

```
MiddleName = NULL     → we don't know the middle name
MiddleName = ''       → the middle name is explicitly empty (a zero-length string)
MiddleName = 'N/A'   → just a string that literally says "N/A"

All three are DIFFERENT values.
```

- Real-world examples of legitimate NULL usage:
  - `EndDate = NULL` → the subscription is still active (no end date yet)
  - `ManagerId = NULL` → this employee is the CEO (no manager above them)
  - `MiddleName = NULL` → we haven't collected the middle name
  - `PhoneNumber = NULL` → the user didn't provide a phone number

```ad-note
title: The Origin of NULL
NULL was introduced by E.F. Codd, the inventor of the relational model, to represent missing or inapplicable information. It is a fundamental part of the relational model, not a hack or afterthought. Every SQL database implements NULL.
```

---

### Three-Valued Logic — TRUE, FALSE, UNKNOWN

- In most programming languages, boolean logic has two values: `TRUE` and `FALSE`. SQL has **three**: `TRUE`, `FALSE`, and `UNKNOWN`.
- Any comparison involving NULL produces `UNKNOWN` — not TRUE, not FALSE:

```sql
-- Assume Age is NULL for a particular row:

Age > 25        → UNKNOWN    (is an unknown value greater than 25? we don't know)
Age = 25        → UNKNOWN    (is an unknown value equal to 25? we don't know)
Age <> 25       → UNKNOWN    (is an unknown value not equal to 25? we don't know)
Age = NULL      → UNKNOWN    (is an unknown value equal to another unknown? STILL unknown!)
NULL = NULL     → UNKNOWN    (two unknowns are not "equal" — they're just both unknown)
```

- SQL `WHERE` clauses only return rows where the condition evaluates to `TRUE`. Rows where the condition is `FALSE` or `UNKNOWN` are **excluded**.
- This is why NULL comparisons are so tricky — they silently exclude rows.

---

### The Cardinal Rule: IS NULL / IS NOT NULL

```ad-warning
title: Never Use = NULL or != NULL
This is the single most common NULL mistake:

- `WHERE Age = NULL` → returns **no rows** (always evaluates to UNKNOWN)
- `WHERE Age != NULL` → returns **no rows** (always evaluates to UNKNOWN)
- `WHERE Age <> NULL` → returns **no rows** (always evaluates to UNKNOWN)

Always use:
- `WHERE Age IS NULL` → returns rows where Age has no value
- `WHERE Age IS NOT NULL` → returns rows where Age has a value
```

```sql
-- WRONG — returns nothing, even if NULLs exist
SELECT * FROM Users WHERE MiddleName = NULL;

-- CORRECT — returns rows where MiddleName is NULL
SELECT * FROM Users WHERE MiddleName IS NULL;

-- CORRECT — returns rows where MiddleName has a value
SELECT * FROM Users WHERE MiddleName IS NOT NULL;
```

---

### NULL in Arithmetic and String Operations

- Any arithmetic or string operation involving NULL produces NULL — NULL propagates through expressions:

```sql
5 + NULL          = NULL        -- any math with NULL = NULL
10 * NULL         = NULL
NULL / 0          = NULL        -- not a division-by-zero error!
100 - NULL + 50   = NULL        -- the entire expression is NULL

'Hello' + NULL    = NULL        -- SQL Server: string concat with NULL = NULL
CONCAT('Hello', NULL) = 'Hello' -- CONCAT function treats NULL as empty string (varies by DBMS)
```

```ad-note
title: CONCAT vs + for String Concatenation
In SQL Server, the `+` operator propagates NULL: `'Hello' + NULL = NULL`.
The `CONCAT()` function treats NULL as an empty string: `CONCAT('Hello', NULL) = 'Hello'`.

In MySQL, `CONCAT()` returns NULL if any argument is NULL. Use `CONCAT_WS()` (concat with separator) or `IFNULL()` to handle NULLs.

Be aware of this difference when writing cross-platform SQL.
```

---

### NULL in Boolean Logic

- Three-valued logic truth tables — these are essential for understanding compound `WHERE` conditions:

#### AND Truth Table

| | TRUE | FALSE | UNKNOWN |
| --- | --- | --- | --- |
| **TRUE** | TRUE | FALSE | UNKNOWN |
| **FALSE** | FALSE | FALSE | FALSE |
| **UNKNOWN** | UNKNOWN | FALSE | UNKNOWN |

- Key insight: `UNKNOWN AND TRUE = UNKNOWN`, `UNKNOWN AND FALSE = FALSE`

#### OR Truth Table

| | TRUE | FALSE | UNKNOWN |
| --- | --- | --- | --- |
| **TRUE** | TRUE | TRUE | TRUE |
| **FALSE** | TRUE | FALSE | UNKNOWN |
| **UNKNOWN** | TRUE | UNKNOWN | UNKNOWN |

- Key insight: `UNKNOWN OR TRUE = TRUE`, `UNKNOWN OR FALSE = UNKNOWN`

#### NOT Truth Table

| Input | NOT |
| --- | --- |
| TRUE | FALSE |
| FALSE | TRUE |
| UNKNOWN | UNKNOWN |

- Key insight: `NOT UNKNOWN = UNKNOWN` — negating an unknown doesn't make it known.

```ad-warning
title: Practical Trap — NOT IN with NULLs
One of the most dangerous NULL pitfalls:

- `WHERE Id IN (1, 2, NULL)` — works "as expected" (returns rows where Id is 1 or 2; NULL comparison produces UNKNOWN, which is harmless here)
- `WHERE Id NOT IN (1, 2, NULL)` — returns **NO ROWS AT ALL**

Why? `NOT IN (1, 2, NULL)` expands to `Id <> 1 AND Id <> 2 AND Id <> NULL`. The last comparison is always UNKNOWN, making the entire AND expression UNKNOWN for every row.

**Fix**: Use `NOT IN` only with subqueries or lists that are guaranteed to have no NULLs, or use `NOT EXISTS` instead.
```

---

### NULL in Aggregate Functions

- Aggregate functions handle NULLs in specific, well-defined ways:

```sql
-- Sample data:
-- Scores table: 90, 80, NULL, 70, NULL

SELECT COUNT(*)       FROM Scores;  -- 5 (counts ALL rows, including NULLs)
SELECT COUNT(Score)   FROM Scores;  -- 3 (counts only NON-NULL values)
SELECT SUM(Score)     FROM Scores;  -- 240 (sums non-NULL values: 90 + 80 + 70)
SELECT AVG(Score)     FROM Scores;  -- 80 (240 / 3, not 240 / 5!)
SELECT MIN(Score)     FROM Scores;  -- 70 (ignores NULLs)
SELECT MAX(Score)     FROM Scores;  -- 90 (ignores NULLs)
```

```ad-important
title: COUNT(*) vs COUNT(column)
This distinction is critical:

- `COUNT(*)` counts **all rows**, regardless of NULL values in any column
- `COUNT(column)` counts only rows where that specific column is **not NULL**

If a table has 1000 rows but 200 have `Email = NULL`:
- `COUNT(*)` = 1000
- `COUNT(Email)` = 800
```

```ad-warning
title: AVG and NULLs — A Common Source of Incorrect Reports
`AVG()` ignores NULLs entirely. This means the denominator is the count of non-NULL values, not the total row count.

Example: Students' test scores — some students didn't take the test (NULL score).
- Scores: 90, 80, NULL, NULL, 70
- `AVG(Score)` = (90 + 80 + 70) / 3 = **80**
- But if NULL means "score of 0" (absent students get zero), the real average is (90 + 80 + 0 + 0 + 70) / 5 = **48**

If NULLs should be treated as a specific value, use `COALESCE`:
- `AVG(COALESCE(Score, 0))` = (90 + 80 + 0 + 0 + 70) / 5 = 48
```

---

### Handling NULLs — COALESCE, ISNULL, IFNULL

- These functions replace NULL with a default value:

#### COALESCE (Standard SQL — works everywhere)

- Returns the **first non-NULL** value from a list of arguments:

```sql
SELECT COALESCE(MiddleName, 'N/A') FROM Users;
-- If MiddleName is NULL → 'N/A'
-- If MiddleName is 'James' → 'James'

-- Multiple fallbacks:
SELECT COALESCE(PreferredName, FirstName, 'Unknown') FROM Users;
-- Returns PreferredName if not NULL,
-- else FirstName if not NULL,
-- else 'Unknown'
```

#### ISNULL (SQL Server specific)

```sql
SELECT ISNULL(MiddleName, 'N/A') FROM Users;
-- Same as COALESCE for two arguments, but SQL Server only
```

#### IFNULL (MySQL / MariaDB specific)

```sql
SELECT IFNULL(MiddleName, 'N/A') FROM Users;
-- Same as ISNULL but for MySQL/MariaDB
```

```ad-tip
title: Prefer COALESCE over ISNULL/IFNULL
`COALESCE` is standard SQL and works across all databases. It also accepts multiple arguments (chained fallbacks), while `ISNULL` and `IFNULL` accept only two.

There is one subtle difference in SQL Server: `ISNULL` returns the data type of the first argument, while `COALESCE` follows standard type precedence rules. This rarely matters in practice, but can cause unexpected truncation:

- `ISNULL(CAST(NULL AS VARCHAR(5)), 'This is a long string')` → `'This '` (truncated to 5 chars)
- `COALESCE(CAST(NULL AS VARCHAR(5)), 'This is a long string')` → `'This is a long string'` (full string)
```

---

### NULLIF — The Reverse of COALESCE

- `NULLIF(a, b)` returns NULL if `a = b`, otherwise returns `a`. Useful for preventing division by zero:

```sql
-- Without NULLIF: division by zero error if TotalHours = 0
SELECT TotalPay / TotalHours FROM Employees;

-- With NULLIF: returns NULL instead of error when TotalHours = 0
SELECT TotalPay / NULLIF(TotalHours, 0) FROM Employees;
-- If TotalHours = 0, NULLIF returns NULL, and TotalPay / NULL = NULL (no error)
```

---

### NULL and NOT NULL Constraints

- When defining a column, you specify whether it accepts NULLs:

```sql
CREATE TABLE Users (
    Id         INT PRIMARY KEY IDENTITY,
    Name       NVARCHAR(100) NOT NULL,      -- required — must have a value
    Email      VARCHAR(200) NOT NULL,        -- required
    MiddleName NVARCHAR(50) NULL,            -- optional — NULL is the default
    Phone      VARCHAR(20)                   -- NULL is the default if not specified
);
```

- `NOT NULL` — the column must have a value on every row. Inserts without this column will fail.
- `NULL` (or omitting the keyword) — the column allows NULL values. This is the default behavior.

```ad-important
title: Best Practice — Default to NOT NULL
Make columns `NOT NULL` by default. Only allow `NULL` when "no value" is a **meaningful and intentional state**.

Ask yourself: "Does it make sense for this column to have no value?"
- `Name` → No. Every user has a name. → `NOT NULL`
- `Email` → Depends on your business rules. Usually `NOT NULL`.
- `MiddleName` → Yes. Not everyone has a middle name. → `NULL`
- `EndDate` → Yes. A NULL end date means "still active." → `NULL`
- `CreatedAt` → No. Every row was created at some point. → `NOT NULL DEFAULT GETDATE()`

The more `NOT NULL` columns you have, the fewer NULL-related bugs you'll encounter. NULLs add complexity to every query that touches the column.
```

---

### NULL in DISTINCT, GROUP BY, and ORDER BY

- `DISTINCT` treats all NULLs as the **same value** — multiple NULLs collapse to one NULL in the result:

```sql
SELECT DISTINCT Department FROM Employees;
-- If 3 employees have Department = NULL, only one NULL row appears
```

- `GROUP BY` also treats all NULLs as one group:

```sql
SELECT Department, COUNT(*) FROM Employees GROUP BY Department;
-- All employees with Department = NULL are grouped together
```

- `ORDER BY` places NULLs either first or last, depending on the DBMS:
  - **SQL Server**: NULLs sort **first** (treated as the lowest possible value)
  - **PostgreSQL**: NULLs sort **last** by default (can be changed with `NULLS FIRST` / `NULLS LAST`)
  - **MySQL**: NULLs sort **first** in ascending order

---

### NULL in CASE Expressions

- To check for NULL in a `CASE` expression, use `IS NULL`, not `= NULL`:

```sql
-- CORRECT
SELECT
    Name,
    CASE
        WHEN MiddleName IS NULL THEN 'No middle name'
        ELSE MiddleName
    END AS DisplayMiddleName
FROM Users;

-- WRONG — the WHEN clause is always UNKNOWN, so it always hits ELSE
SELECT
    Name,
    CASE
        WHEN MiddleName = NULL THEN 'No middle name'  -- never matches!
        ELSE MiddleName
    END AS DisplayMiddleName
FROM Users;
```

---

### NULL in Joins

- NULL FK values affect join behavior:
  - An `INNER JOIN` **excludes** rows where the FK is NULL (no match in the parent table)
  - A `LEFT JOIN` **includes** them (with NULLs in all columns from the right table)

```sql
-- Employees with ManagerId = NULL (CEO) will NOT appear in an INNER JOIN:
SELECT e.Name, m.Name AS Manager
FROM Employees e
INNER JOIN Employees m ON e.ManagerId = m.Id;
-- CEO is excluded because NULL = any Id is UNKNOWN

-- Use LEFT JOIN to include them:
SELECT e.Name, COALESCE(m.Name, 'No Manager') AS Manager
FROM Employees e
LEFT JOIN Employees m ON e.ManagerId = m.Id;
-- CEO appears with Manager = 'No Manager'
```

---

### Summary — NULL Rules to Memorize

```ad-tip
title: Quick Reference — How NULL Behaves Everywhere
1. `NULL = NULL` → UNKNOWN (not TRUE)
2. `NULL <> NULL` → UNKNOWN (not TRUE)
3. Always use `IS NULL` / `IS NOT NULL` — never `= NULL`
4. Any arithmetic with NULL → NULL
5. Any string concatenation with NULL → NULL (except `CONCAT()` in some DBMS)
6. `NOT UNKNOWN` → UNKNOWN
7. `COUNT(*)` counts all rows; `COUNT(column)` skips NULLs
8. `SUM`, `AVG`, `MIN`, `MAX` ignore NULLs
9. `AVG` denominator is non-NULL count, not total rows
10. `NOT IN (list with NULL)` → returns no rows
11. Use `COALESCE(column, default)` to replace NULLs
12. Use `NULLIF(a, b)` to turn a specific value into NULL
13. Default to `NOT NULL` — only allow NULL when it's meaningful
```

---

**Next:** See the [[SQL Essentials]] folder for querying data with SELECT, WHERE, JOINs, and more.
