---
tags: [sql, where, filtering]
---

- The `WHERE` clause **filters rows** before they reach the rest of the query. It's how you ask for specific data instead of getting every row in the table.
- `WHERE` executes early in the [[01 - SELECT and FROM#Query Execution Order|query execution order]] — right after `FROM` / `JOIN`, before `GROUP BY`, `HAVING`, or `SELECT`.

---

## Basic Syntax

```sql
SELECT column1, column2
FROM table_name
WHERE condition;
```

```sql
-- Get employees in the Engineering department
SELECT first_name, last_name, salary
FROM employees
WHERE department = 'Engineering';
```

---

## Comparison Operators

| Operator | Meaning | Example |
| --- | --- | --- |
| `=` | Equal to | `WHERE age = 30` |
| `<>` or `!=` | Not equal to | `WHERE status <> 'inactive'` |
| `<` | Less than | `WHERE price < 100` |
| `>` | Greater than | `WHERE salary > 50000` |
| `<=` | Less than or equal | `WHERE quantity <= 10` |
| `>=` | Greater than or equal | `WHERE rating >= 4.5` |

```ad-note
SQL uses `=` for comparison, not `==`. The `<>` operator is the SQL standard for "not equal." MySQL also accepts `!=`, and SQL Server supports both. Prefer `<>` for portability.
```

---

## Logical Operators: AND, OR, NOT

### AND — Both conditions must be true

```sql
SELECT first_name, salary, department
FROM employees
WHERE department = 'Engineering' AND salary > 60000;
```

### OR — At least one condition must be true

```sql
SELECT first_name, department
FROM employees
WHERE department = 'Engineering' OR department = 'Marketing';
```

### NOT — Negates a condition

```sql
SELECT first_name, department
FROM employees
WHERE NOT department = 'Sales';
```

### Operator Precedence

```ad-warning
`AND` has **higher precedence** than `OR`. This is one of the most common sources of bugs in SQL queries. Without parentheses, the query does not do what you'd expect.
```

Consider this query:

```sql
-- WRONG: finds all engineers OR anyone with salary > 80000
SELECT * FROM employees
WHERE department = 'Engineering' OR department = 'Marketing'
  AND salary > 80000;
```

Because `AND` binds first, this actually means:

```sql
-- What the engine sees:
WHERE department = 'Engineering' 
   OR (department = 'Marketing' AND salary > 80000)
```

- This returns **all** engineers (regardless of salary) plus only high-paid marketers.

**Fix it with parentheses:**

```sql
-- CORRECT: engineers or marketers, but only if salary > 80000
SELECT * FROM employees
WHERE (department = 'Engineering' OR department = 'Marketing')
  AND salary > 80000;
```

```ad-important
**Always use parentheses** when combining `AND` and `OR`. Even if you remember the precedence, the next person reading your query might not.
```

---

## BETWEEN — Range Filtering

- `BETWEEN` tests whether a value falls within a range, **inclusive** of both endpoints.

```sql
-- Employees earning between 40000 and 70000 (inclusive)
SELECT first_name, salary
FROM employees
WHERE salary BETWEEN 40000 AND 70000;
```

- This is equivalent to:

```sql
WHERE salary >= 40000 AND salary <= 70000
```

- Works with dates too:

```sql
-- Orders placed in January 2025
SELECT order_id, order_date, total
FROM orders
WHERE order_date BETWEEN '2025-01-01' AND '2025-01-31';
```

```ad-warning
Be careful with `BETWEEN` on datetime columns. `BETWEEN '2025-01-01' AND '2025-01-31'` includes midnight on January 31st but **excludes** anything after midnight (like `2025-01-31 08:30:00` if the column has time precision). For datetime ranges, prefer:
`WHERE order_date >= '2025-01-01' AND order_date < '2025-02-01'`
This pattern is called a **half-open interval** and is immune to time-precision surprises.
```

- `NOT BETWEEN` excludes the range:

```sql
WHERE salary NOT BETWEEN 40000 AND 70000
-- salary < 40000 OR salary > 70000
```

---

## IN — Match Against a List

- `IN` checks whether a value matches any value in a list. It's cleaner than chaining multiple `OR` conditions.

```sql
-- These are equivalent:
SELECT * FROM employees
WHERE department IN ('Engineering', 'Marketing', 'Sales');

SELECT * FROM employees
WHERE department = 'Engineering' 
   OR department = 'Marketing' 
   OR department = 'Sales';
```

- `IN` can also take a subquery (covered in [[06 - Subqueries and Common Table Expressions]]):

```sql
SELECT first_name FROM employees
WHERE department_id IN (SELECT id FROM departments WHERE active = 1);
```

- `NOT IN` excludes the listed values:

```sql
SELECT * FROM employees
WHERE department NOT IN ('HR', 'Legal');
```

```ad-warning
`NOT IN` has a dangerous trap with `NULL` values. If **any** value in the list is `NULL`, the entire `NOT IN` returns no rows. This happens because of [[06 - NULL and Three-Valued Logic|three-valued logic]]: comparing anything to `NULL` yields `UNKNOWN`, and `NOT UNKNOWN` is still `UNKNOWN`.

```sql
-- This returns NO rows if any department is NULL:
SELECT * FROM employees
WHERE department NOT IN ('Sales', NULL);
```

Use `NOT EXISTS` instead of `NOT IN` when the subquery might contain NULLs. See [[06 - Subqueries and Common Table Expressions]].
```

---

## LIKE — Pattern Matching

- `LIKE` matches strings against a pattern using two wildcards:

| Wildcard | Meaning | Example |
| --- | --- | --- |
| `%` | Any sequence of characters (zero or more) | `'J%'` matches "John", "Jane", "J" |
| `_` | Exactly one character | `'A_B'` matches "AXB", "A1B", not "AB" or "AXXB" |

### Common Patterns

```sql
-- Names starting with 'J'
SELECT * FROM employees WHERE first_name LIKE 'J%';

-- Names ending with 'son'
SELECT * FROM employees WHERE last_name LIKE '%son';

-- Names containing 'mid' anywhere
SELECT * FROM employees WHERE last_name LIKE '%mid%';

-- Names that are exactly 4 characters
SELECT * FROM employees WHERE first_name LIKE '____';

-- Second character is 'a'
SELECT * FROM employees WHERE first_name LIKE '_a%';
```

### Case Sensitivity

- **SQL Server**: `LIKE` is case-**insensitive** by default (depends on collation).
- **MySQL/MariaDB**: `LIKE` is case-**insensitive** by default for `VARCHAR` columns.
- **PostgreSQL**: `LIKE` is case-**sensitive**. Use `ILIKE` for case-insensitive matching.

### Escaping Wildcards

- To search for a literal `%` or `_`, escape them:

```sql
-- SQL Server and MySQL
SELECT * FROM products WHERE name LIKE '%10\%%' ESCAPE '\';
-- Finds names containing "10%"

-- Alternative (SQL Server)
SELECT * FROM products WHERE name LIKE '%10[%]%';
```

- `NOT LIKE` negates the pattern:

```sql
SELECT * FROM employees WHERE email NOT LIKE '%@gmail.com';
```

```ad-note
`LIKE` patterns that start with `%` (e.g., `'%son'`) **cannot use indexes** and force a full table scan. If possible, restructure the query or use full-text search instead. Patterns that start with a literal character (e.g., `'J%'`) can use indexes efficiently.
```

---

## IS NULL / IS NOT NULL

- In SQL, `NULL` is not a value — it represents the **absence of a value**. You cannot use `=` or `<>` to compare against `NULL`; those always return `UNKNOWN`.

```sql
-- WRONG: this will never match any rows
SELECT * FROM employees WHERE manager_id = NULL;

-- CORRECT: use IS NULL
SELECT * FROM employees WHERE manager_id IS NULL;

-- Find rows that DO have a value
SELECT * FROM employees WHERE manager_id IS NOT NULL;
```

```ad-important
`NULL = NULL` is **not** true in SQL. It evaluates to `UNKNOWN`. Always use `IS NULL` or `IS NOT NULL`. This is fundamental to SQL's [[06 - NULL and Three-Valued Logic|three-valued logic]] system.
```

---

## Combining Multiple Conditions

- Real-world queries often combine many conditions. Use parentheses to make the logic clear.

```sql
-- Active employees in Engineering or Marketing, earning over 50K,
-- who were hired in the last 2 years
SELECT first_name, last_name, department, salary, hire_date
FROM employees
WHERE active = 1
  AND (department = 'Engineering' OR department = 'Marketing')
  AND salary > 50000
  AND hire_date >= '2024-01-01';
```

- Equivalent using `IN` (cleaner for the department check):

```sql
SELECT first_name, last_name, department, salary, hire_date
FROM employees
WHERE active = 1
  AND department IN ('Engineering', 'Marketing')
  AND salary > 50000
  AND hire_date >= '2024-01-01';
```

---

## Filtering with Expressions

- You can use expressions and functions in `WHERE`:

```sql
-- Employees whose annual salary exceeds 100K
SELECT first_name, salary
FROM employees
WHERE salary * 12 > 100000;

-- Orders placed on a Monday (MySQL)
SELECT * FROM orders
WHERE DAYOFWEEK(order_date) = 2;

-- Orders placed on a Monday (SQL Server)
SELECT * FROM orders
WHERE DATEPART(WEEKDAY, order_date) = 2;
```

```ad-warning
Applying a function to a column in `WHERE` (like `WHERE YEAR(hire_date) = 2025`) prevents index usage on that column. The database must evaluate the function on every row — a full table scan. This is called a **non-sargable** predicate. Prefer range-based conditions instead:

```sql
-- Slow (non-sargable — can't use index on hire_date)
WHERE YEAR(hire_date) = 2025

-- Fast (sargable — index on hire_date is used)
WHERE hire_date >= '2025-01-01' AND hire_date < '2026-01-01'
```
```

---

## Performance Considerations

```ad-note
A few rules of thumb for writing efficient `WHERE` clauses:
- **Indexed columns are fast.** `WHERE` conditions on indexed columns allow the database to skip most rows using the index rather than scanning the entire table.
- **Non-indexed columns cause full table scans.** Every row must be checked.
- **Avoid functions on columns** — use them on the value side instead (`WHERE hire_date >= '2025-01-01'` not `WHERE YEAR(hire_date) = 2025`).
- **`LIKE` with a leading wildcard** (`'%value'`) cannot use indexes.
- **`OR` can prevent index usage.** Sometimes rewriting as `UNION ALL` is faster.

Performance tuning is covered in depth in the [[Performance and Administration]] folder.
```

---

## Quick Reference

| Need | Syntax |
| --- | --- |
| Exact match | `WHERE col = 'value'` |
| Not equal | `WHERE col <> 'value'` |
| Range (inclusive) | `WHERE col BETWEEN 10 AND 20` |
| In a list | `WHERE col IN ('a', 'b', 'c')` |
| Pattern match | `WHERE col LIKE 'J%'` |
| Is missing | `WHERE col IS NULL` |
| Combine | `WHERE a = 1 AND (b = 2 OR b = 3)` |
| Negate | `WHERE NOT col = 'x'` or `WHERE col NOT IN (...)` |

```ad-note
Now that you can read and filter data, learn how to modify it: [[03 - INSERT, UPDATE, DELETE]].
```
