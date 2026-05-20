---
tags: [sql, functions, date]
---

- Date/time functions are heavily dialect-dependent. This note covers the most common operations with MySQL, PostgreSQL, and SQL Server syntax.

---

### Current Date and Time

```sql
-- Current date + time:
SELECT NOW();                  -- MySQL, PostgreSQL
SELECT CURRENT_TIMESTAMP;      -- Standard SQL (all RDBMS)
SELECT GETDATE();              -- SQL Server

-- Current date only:
SELECT CURDATE();              -- MySQL
SELECT CURRENT_DATE;           -- PostgreSQL, Standard SQL
SELECT CAST(GETDATE() AS DATE); -- SQL Server
```

---

### Extract Components

```sql
-- MySQL:
SELECT YEAR(hire_date), MONTH(hire_date), DAY(hire_date) FROM employees;

-- PostgreSQL:
SELECT EXTRACT(YEAR FROM hire_date), EXTRACT(MONTH FROM hire_date) FROM employees;

-- SQL Server:
SELECT YEAR(hire_date), MONTH(hire_date), DAY(hire_date) FROM employees;
-- or: DATEPART(YEAR, hire_date)
```

---

### Date Arithmetic

```sql
-- MySQL:
SELECT DATE_ADD(hire_date, INTERVAL 30 DAY) FROM employees;   -- add 30 days
SELECT DATE_SUB(hire_date, INTERVAL 1 MONTH) FROM employees;  -- subtract 1 month

-- PostgreSQL:
SELECT hire_date + INTERVAL '30 days' FROM employees;
SELECT hire_date - INTERVAL '1 month' FROM employees;

-- SQL Server:
SELECT DATEADD(DAY, 30, hire_date) FROM employees;
SELECT DATEADD(MONTH, -1, hire_date) FROM employees;
```

---

### Date Difference

```sql
-- MySQL (returns days):
SELECT DATEDIFF(end_date, start_date) FROM projects;

-- PostgreSQL (subtract directly):
SELECT end_date - start_date FROM projects;  -- returns integer (days)

-- SQL Server:
SELECT DATEDIFF(DAY, start_date, end_date) FROM projects;
SELECT DATEDIFF(MONTH, start_date, end_date) FROM projects;
```

---

### Date Formatting

```sql
-- MySQL:
SELECT DATE_FORMAT(hire_date, '%Y-%m-%d') FROM employees;  -- '2024-01-15'
SELECT DATE_FORMAT(hire_date, '%M %d, %Y') FROM employees;  -- 'January 15, 2024'

-- PostgreSQL:
SELECT TO_CHAR(hire_date, 'YYYY-MM-DD') FROM employees;

-- SQL Server:
SELECT FORMAT(hire_date, 'yyyy-MM-dd') FROM employees;
SELECT CONVERT(VARCHAR, hire_date, 23) FROM employees;  -- ISO format
```

---

### String to Date

```sql
-- MySQL:
SELECT STR_TO_DATE('15/01/2024', '%d/%m/%Y');

-- PostgreSQL:
SELECT TO_DATE('15/01/2024', 'DD/MM/YYYY');

-- SQL Server:
SELECT CONVERT(DATE, '2024-01-15', 23);
```

```ad-warning
**Always store dates as DATE/DATETIME types, never as strings.** String dates break sorting, comparisons, and date arithmetic. If you receive dates as strings from an application, convert them on insert. See [[Data Types]].
```
