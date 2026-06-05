---
tags: [sql, aggregation, grouping]
---

- **Aggregate functions** collapse many rows into a single summary value — counts, totals, averages. `GROUP BY` splits the data into groups so each group gets its own summary.
- This is how you go from raw data to answers like "how many orders per customer?", "what's the average salary per department?", or "which product generated the most revenue?"

---

## Aggregate Functions

- Aggregate functions operate on a **set of rows** and return a single value.

| Function | Returns | NULL Handling |
| --- | --- | --- |
| `COUNT(*)` | Total number of rows | Counts all rows, including those with NULLs |
| `COUNT(column)` | Number of non-NULL values in that column | Ignores NULLs |
| `COUNT(DISTINCT column)` | Number of unique non-NULL values | Ignores NULLs |
| `SUM(column)` | Sum of all values | Ignores NULLs |
| `AVG(column)` | Average of all values | Ignores NULLs |
| `MIN(column)` | Smallest value | Ignores NULLs |
| `MAX(column)` | Largest value | Ignores NULLs |

```ad-warning
**`COUNT(*)` vs `COUNT(column)` is a classic trap.** `COUNT(*)` counts all rows. `COUNT(column)` counts only rows where that column is not NULL. If your column has NULLs, these return different numbers.

```sql
-- Table has 100 rows, but 10 have NULL email
SELECT COUNT(*) FROM users;          -- 100
SELECT COUNT(email) FROM users;      -- 90
SELECT COUNT(DISTINCT email) FROM users; -- might be even fewer
```
```

### Aggregates Without GROUP BY

- Without `GROUP BY`, the aggregate runs on the **entire table** as one group:

```sql
SELECT COUNT(*) AS TotalEmployees FROM employees;
-- Returns: 150

SELECT AVG(salary) AS AvgSalary FROM employees;
-- Returns: 65432.50

SELECT MIN(hire_date) AS EarliestHire, MAX(hire_date) AS LatestHire
FROM employees;
-- Returns: 2015-03-12, 2026-01-15
```

---

## GROUP BY — Split into Groups

- `GROUP BY` partitions the rows into groups based on one or more columns. Each group then gets its own aggregate value.

```sql
SELECT department, COUNT(*) AS EmployeeCount
FROM employees
GROUP BY department;
```

**Result:**

| department | EmployeeCount |
| --- | --- |
| Engineering | 45 |
| Marketing | 20 |
| Sales | 35 |
| HR | 15 |

### How GROUP BY Works — Step by Step

Consider this query:

```sql
SELECT department, AVG(salary) AS AvgSalary
FROM employees
GROUP BY department;
```

1. The engine looks at all rows in `employees`.
2. It groups them by `department` — all Engineering rows together, all Marketing rows together, etc.
3. For each group, it computes `AVG(salary)`.
4. It returns one row per group.

### The Golden Rule of GROUP BY

```ad-important
Every column in `SELECT` must either be:
1. Listed in `GROUP BY`, **or**
2. Inside an aggregate function (`COUNT`, `SUM`, `AVG`, etc.)

You **cannot** select a non-aggregated column that isn't in `GROUP BY`. The database wouldn't know which value from the group to show.

```sql
-- ERROR: first_name is not in GROUP BY and not aggregated
SELECT department, first_name, COUNT(*)
FROM employees
GROUP BY department;

-- FIX: either add first_name to GROUP BY or remove it
SELECT department, COUNT(*) FROM employees GROUP BY department;
```
```

```ad-note
MySQL has a non-standard behavior: with `ONLY_FULL_GROUP_BY` mode disabled (the old default), it allows non-aggregated columns not in `GROUP BY`, picking an arbitrary value. This is **almost always a bug**. Modern MySQL (5.7.5+) enables `ONLY_FULL_GROUP_BY` by default, matching the standard behavior. SQL Server always enforces this rule strictly.
```

### GROUP BY Multiple Columns

```sql
-- Count employees per department AND job title
SELECT department, job_title, COUNT(*) AS EmployeeCount
FROM employees
GROUP BY department, job_title
ORDER BY department, EmployeeCount DESC;
```

**Result:**

| department | job_title | EmployeeCount |
| --- | --- | --- |
| Engineering | Senior Developer | 15 |
| Engineering | Junior Developer | 12 |
| Engineering | Tech Lead | 5 |
| Marketing | Content Writer | 10 |
| Marketing | Designer | 8 |

---

## HAVING — Filter Groups

- `WHERE` filters **individual rows** before grouping. `HAVING` filters **groups** after aggregation.

```sql
-- Departments with more than 20 employees
SELECT department, COUNT(*) AS EmployeeCount
FROM employees
GROUP BY department
HAVING COUNT(*) > 20;
```

```sql
-- Departments where average salary exceeds 70K
SELECT department, AVG(salary) AS AvgSalary
FROM employees
GROUP BY department
HAVING AVG(salary) > 70000;
```

### WHERE vs HAVING

| | WHERE | HAVING |
| --- | --- | --- |
| **Filters** | Individual rows | Groups (after aggregation) |
| **Executes** | Before `GROUP BY` | After `GROUP BY` |
| **Can use aggregates?** | No | Yes |
| **Can reference columns?** | Any column in the table | Only grouped or aggregated columns |

```sql
-- WHERE and HAVING together: different roles
SELECT department, AVG(salary) AS AvgSalary
FROM employees
WHERE active = 1               -- Step 1: keep only active employees
GROUP BY department             -- Step 2: group by department
HAVING AVG(salary) > 60000     -- Step 3: keep groups with avg > 60K
ORDER BY AvgSalary DESC;       -- Step 4: sort the result
```

```ad-warning
A common mistake is using `WHERE` with aggregate functions:

```sql
-- WRONG: can't use COUNT in WHERE
SELECT department, COUNT(*)
FROM employees
WHERE COUNT(*) > 10
GROUP BY department;

-- CORRECT: use HAVING for aggregate conditions
SELECT department, COUNT(*)
FROM employees
GROUP BY department
HAVING COUNT(*) > 10;
```
```

```ad-note
**Performance tip**: always prefer `WHERE` over `HAVING` when possible. `WHERE` eliminates rows before the grouping happens, so the engine has less data to group. `HAVING` only removes groups after the expensive aggregation work is already done. If a condition doesn't involve an aggregate function, put it in `WHERE`.
```

---

## GROUP BY with JOIN

- Combining JOINs with GROUP BY is where these concepts become truly powerful. You join tables to bring data together, then group and aggregate the combined result.

```sql
-- How many orders and how much has each user spent?
SELECT 
    u.Name,
    COUNT(o.OrderId) AS OrderCount,
    COALESCE(SUM(o.Total), 0) AS TotalSpent
FROM Users u
LEFT JOIN Orders o ON u.Id = o.UserId
GROUP BY u.Name
ORDER BY TotalSpent DESC;
```

**Result:**

| Name | OrderCount | TotalSpent |
| --- | --- | --- |
| Alice | 5 | 2340.00 |
| Bob | 3 | 850.00 |
| Carol | 0 | 0.00 |

- `LEFT JOIN` ensures Carol appears even with zero orders.
- `COALESCE(SUM(o.Total), 0)` turns the NULL sum (from no orders) into 0.
- `COUNT(o.OrderId)` instead of `COUNT(*)` correctly returns 0 for Carol — `COUNT(*)` would return 1 because the LEFT JOIN still produces one row for Carol.

```ad-important
When using `LEFT JOIN` with `GROUP BY`, use `COUNT(right_table.column)` not `COUNT(*)`. `COUNT(*)` counts the joined row (which exists as a NULL-padded row) and returns 1 instead of 0 for unmatched rows.
```

### More Real-World Examples

```sql
-- Revenue per product category, only categories with > $10K
SELECT 
    c.CategoryName,
    COUNT(oi.OrderItemId) AS ItemsSold,
    SUM(oi.Quantity * oi.UnitPrice) AS Revenue
FROM Categories c
INNER JOIN Products p ON c.Id = p.CategoryId
INNER JOIN OrderItems oi ON p.ProductId = oi.ProductId
GROUP BY c.CategoryName
HAVING SUM(oi.Quantity * oi.UnitPrice) > 10000
ORDER BY Revenue DESC;
```

```sql
-- Monthly order summary for the current year
SELECT 
    YEAR(o.OrderDate) AS OrderYear,
    MONTH(o.OrderDate) AS OrderMonth,
    COUNT(*) AS OrderCount,
    SUM(o.Total) AS MonthlyRevenue,
    AVG(o.Total) AS AvgOrderValue
FROM Orders o
WHERE o.OrderDate >= '2026-01-01'
GROUP BY YEAR(o.OrderDate), MONTH(o.OrderDate)
ORDER BY OrderYear, OrderMonth;
```

---

## Full Query Execution Order — Everything Comes Together

Now that you know `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `SELECT`, and `ORDER BY`, the full execution order makes complete sense:

| Step | Clause | What Happens |
| --- | --- | --- |
| 1 | `FROM` / `JOIN` | Identify source tables, combine them |
| 2 | `WHERE` | Filter individual rows |
| 3 | `GROUP BY` | Group remaining rows |
| 4 | `HAVING` | Filter groups |
| 5 | `SELECT` | Compute output columns and aliases |
| 6 | `DISTINCT` | Remove duplicate rows |
| 7 | `ORDER BY` | Sort the result |
| 8 | `TOP` / `LIMIT` | Restrict row count |

**This is why:**
- `WHERE` can't use aliases (defined in step 5, `WHERE` runs at step 2)
- `WHERE` can't use aggregates (grouping hasn't happened yet at step 2)
- `HAVING` can use aggregates (runs after `GROUP BY` at step 4)
- `ORDER BY` can use aliases (runs after `SELECT` at step 7)
- `HAVING` can't use aliases in SQL Server (step 4 before step 5) — but MySQL allows it as a non-standard extension

```ad-note
This execution order was introduced in [[01 - SELECT and FROM]]. Now with GROUP BY and HAVING in the picture, you can see the full story.
```

---

## GROUP BY with Expressions

- You can group by computed expressions, not just column names:

```sql
-- SQL Server: group by year of hire
SELECT YEAR(hire_date) AS HireYear, COUNT(*) AS Hired
FROM employees
GROUP BY YEAR(hire_date)
ORDER BY HireYear;

-- Group by salary ranges
SELECT 
    CASE 
        WHEN salary < 40000 THEN 'Under 40K'
        WHEN salary BETWEEN 40000 AND 70000 THEN '40K-70K'
        WHEN salary BETWEEN 70001 AND 100000 THEN '70K-100K'
        ELSE 'Over 100K'
    END AS SalaryBand,
    COUNT(*) AS EmployeeCount
FROM employees
GROUP BY 
    CASE 
        WHEN salary < 40000 THEN 'Under 40K'
        WHEN salary BETWEEN 40000 AND 70000 THEN '40K-70K'
        WHEN salary BETWEEN 70001 AND 100000 THEN '70K-100K'
        ELSE 'Over 100K'
    END;
```

```ad-note
In the CASE example above, you must repeat the entire CASE expression in GROUP BY (SQL Server) because aliases aren't available at that stage. MySQL allows grouping by the alias as a shortcut, but it's non-standard.
```

---

## Useful Aggregate Patterns

### Count Distinct

```sql
-- How many unique products has each customer ordered?
SELECT c.Name, COUNT(DISTINCT oi.ProductId) AS UniqueProducts
FROM Customers c
INNER JOIN Orders o ON c.Id = o.CustomerId
INNER JOIN OrderItems oi ON o.OrderId = oi.OrderId
GROUP BY c.Name;
```

### Conditional Aggregation

- Count or sum based on a condition using CASE inside the aggregate:

```sql
-- Count active vs inactive employees per department
SELECT 
    department,
    COUNT(*) AS Total,
    COUNT(CASE WHEN active = 1 THEN 1 END) AS ActiveCount,
    COUNT(CASE WHEN active = 0 THEN 1 END) AS InactiveCount
FROM employees
GROUP BY department;
```

### Finding the Top N per Group

- A common need — "top 3 earners per department" — requires window functions (see [[04 - Advanced Queries]] in the SQL folder) or a correlated subquery ([[06 - Subqueries and Common Table Expressions]]).

```ad-note
Aggregation and grouping unlock the ability to answer business questions. Next, learn how to nest queries and use CTEs for even more complex analysis: [[06 - Subqueries and Common Table Expressions]].
```
