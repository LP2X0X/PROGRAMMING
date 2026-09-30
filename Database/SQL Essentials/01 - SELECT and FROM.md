---
tags: [sql, select, querying]
---

- `SELECT` is how you **read** data from a database. It is the single most frequently used SQL statement. Every report, every dashboard, every data export begins with `SELECT`.
- The `FROM` clause tells SQL **which table** to read from.

```ad-note
`SELECT` is a purely read-only operation. It never modifies data. For modifying data, see [[03 - INSERT, UPDATE, DELETE]].
```

---

## Basic Syntax

```sql
SELECT column1, column2, column3
FROM table_name;
```

- List the columns you want, separated by commas, then specify the table with `FROM`.

```sql
-- Get every employee's name and email
SELECT first_name, last_name, email
FROM employees;
```

---

## SELECT * (All Columns)

```sql
SELECT * FROM employees;
```

- The `*` wildcard means "give me every column." Useful for quick exploration, but **avoid it in production code**:
  - If someone adds columns to the table later, your query returns unexpected data.
  - You transfer more data over the network than you need.
  - The query optimizer can't take advantage of covering indexes.
  - It makes the query harder to read — the reader has to check the schema to know what columns come back.

```ad-note
The from clause defines the tables used by a query, along with the means of linking the tables together.
```

```ad-important
Use `SELECT *` in ad-hoc exploration. In application code, stored procedures, or views, **always list the columns explicitly**.
```

---

## Column Aliases

- Use `AS` to rename a column in the result set. The alias only affects the output — it does not change the actual table.

```sql
SELECT 
    first_name AS Name,
    salary * 12 AS AnnualSalary,
    hire_date AS StartDate
FROM employees;
```

- The `AS` keyword is technically optional — `SELECT first_name Name` works — but always include it for clarity.
- Aliases with spaces or reserved words require quoting:

```sql
-- SQL Server: square brackets or double quotes
SELECT first_name AS [Full Name] FROM employees;

-- MySQL/MariaDB: backticks or double quotes
SELECT first_name AS `Full Name` FROM employees;
```

- Aliases are evaluated late in the execution order (at the `SELECT` phase), so you **cannot** reference an alias in a `WHERE` or `GROUP BY` clause in most databases. More on this in the execution order section below.

---

## DISTINCT — Remove Duplicates

- `DISTINCT` eliminates duplicate rows from the result:

```sql
SELECT DISTINCT department FROM employees;
```

- `DISTINCT` applies to the **entire row**, not just one column. If you write `SELECT DISTINCT department, job_title FROM employees`, it returns unique combinations of department and job_title.

```sql
-- How many unique cities do our customers come from?
SELECT DISTINCT city FROM customers;

-- Unique department/title pairs
SELECT DISTINCT department, job_title FROM employees;
```

```ad-warning
`DISTINCT` can be a performance red flag. If you find yourself using `DISTINCT` to "fix" duplicate rows, the real problem is usually a bad JOIN that's multiplying rows. Fix the JOIN instead of masking it with `DISTINCT`.
```

---

## TOP / LIMIT — Restrict Row Count

- Sometimes you only want the first N rows. The syntax differs between database engines.

**SQL Server — `TOP`:**

```sql
SELECT TOP 10 first_name, last_name, salary
FROM employees
ORDER BY salary DESC;
```

- `TOP` also supports `PERCENT`:

```sql
SELECT TOP 10 PERCENT * FROM employees ORDER BY salary DESC;
```

- `WITH TIES` includes additional rows that tie with the last row:

```sql
SELECT TOP 5 WITH TIES first_name, salary
FROM employees
ORDER BY salary DESC;
-- If 3 people tie for 5th place, you get 7 rows
```

**MySQL/MariaDB — `LIMIT`:**

```sql
SELECT first_name, last_name, salary
FROM employees
ORDER BY salary DESC
LIMIT 10;
```

- `LIMIT` with `OFFSET` for pagination:

```sql
-- Skip the first 20 rows, then return 10
SELECT first_name, salary
FROM employees
ORDER BY salary DESC
LIMIT 10 OFFSET 20;
```

| Feature | SQL Server | MySQL/MariaDB |
| --- | --- | --- |
| Limit rows | `TOP N` | `LIMIT N` |
| Offset | `OFFSET N ROWS FETCH NEXT M ROWS ONLY` (2012+) | `LIMIT M OFFSET N` |
| Percentage | `TOP N PERCENT` | Not supported natively |
| Include ties | `TOP N WITH TIES` | Not supported natively |

```ad-note
SQL Server 2012+ also supports the standard `OFFSET...FETCH` syntax: `ORDER BY salary DESC OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY`. This is preferred for pagination because it's more portable.
```

---

## ORDER BY — Sort Results

- By default, SQL does **not guarantee** any particular row order. If you need results in a specific order, you must use `ORDER BY`.

```sql
SELECT first_name, last_name, salary
FROM employees
ORDER BY salary;
```

- **ASC** (ascending) is the default. **DESC** sorts descending:

```sql
ORDER BY salary DESC;         -- highest first
ORDER BY last_name ASC;       -- A → Z (default)
```

- Sort by multiple columns — the second column breaks ties in the first:

```sql
ORDER BY department ASC, salary DESC;
-- Sort by department A→Z, then within each department, highest salary first
```

- You can `ORDER BY` a column alias:

```sql
SELECT first_name, salary * 12 AS annual_salary
FROM employees
ORDER BY annual_salary DESC;
```

- You can also `ORDER BY` a column's ordinal position (1-based), though this is fragile and generally discouraged:

```sql
SELECT first_name, salary FROM employees
ORDER BY 2 DESC;  -- sorts by the 2nd column (salary)
```

```ad-warning
Never rely on the "natural" order of rows in a table. Without `ORDER BY`, the database engine returns rows in whatever order is most efficient. That order can change after an index rebuild, a statistics update, or even between two identical query executions.
```

---

## SELECT Without FROM

- Some databases allow `SELECT` with no table for quick calculations or system info:

```sql
-- MySQL/MariaDB
SELECT 1 + 1;                    -- Returns 2
SELECT NOW();                    -- Current timestamp
SELECT VERSION();                -- Database version

-- SQL Server
SELECT GETDATE();                -- Current timestamp
SELECT 1 + 1;                    -- Returns 2
SELECT @@VERSION;                -- SQL Server version
```

- In Oracle, you must use the dummy table `dual`: `SELECT 1 + 1 FROM dual;`

---

## Query Execution Order

- SQL does **not** execute in the order you write it. Understanding the true execution order is essential for knowing which clauses can reference which aliases, and why certain errors occur.

| Step | Clause | What It Does |
| --- | --- | --- |
| 1 | `FROM` / `JOIN` | Identify the source table(s) and combine them |
| 2 | `WHERE` | Filter individual rows |
| 3 | `GROUP BY` | Group the remaining rows |
| 4 | `HAVING` | Filter groups |
| 5 | `SELECT` | Choose and compute the output columns |
| 6 | `DISTINCT` | Remove duplicates |
| 7 | `ORDER BY` | Sort the final result |
| 8 | `TOP` / `LIMIT` | Restrict the number of rows returned |

```ad-important
This is why you can't use a column alias in `WHERE` — `WHERE` executes before `SELECT` where the alias is defined. You **can** use an alias in `ORDER BY` because it runs after `SELECT`. This execution order will come up repeatedly in [[02 - WHERE and Filtering]], [[05 - GROUP BY and Aggregation]], and beyond.
```

- What you write:

```sql
SELECT department, COUNT(*) AS emp_count
FROM employees
WHERE active = 1
GROUP BY department
HAVING COUNT(*) > 5
ORDER BY emp_count DESC;
```

- How the engine processes it:

```
1. FROM employees              → load the table
2. WHERE active = 1            → keep only active employees
3. GROUP BY department         → group by department
4. HAVING COUNT(*) > 5         → keep groups with more than 5
5. SELECT department, COUNT(*) → compute the output columns
6. ORDER BY emp_count DESC     → sort by the alias
```

---

## Practical Examples

**Get all products sorted by price, cheapest first:**

```sql
SELECT product_name, price, category
FROM products
ORDER BY price ASC;
```

**Top 5 highest-paid employees:**

```sql
-- SQL Server
SELECT TOP 5 first_name, last_name, salary
FROM employees
ORDER BY salary DESC;

-- MySQL/MariaDB
SELECT first_name, last_name, salary
FROM employees
ORDER BY salary DESC
LIMIT 5;
```

**Unique job titles in the company:**

```sql
SELECT DISTINCT job_title FROM employees ORDER BY job_title;
```

**Rename columns for a cleaner report:**

```sql
SELECT 
    first_name AS [First Name],
    last_name AS [Last Name],
    salary AS [Monthly Salary],
    salary * 12 AS [Annual Salary]
FROM employees
ORDER BY [Annual Salary] DESC;
```

```ad-note
This note covers the fundamentals of reading data. Next, learn how to filter those results with [[02 - WHERE and Filtering]].
```
