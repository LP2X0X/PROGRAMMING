---
tags: [sql, functions, string]
---

- The most commonly used string manipulation functions. Syntax varies by RDBMS — differences noted where they matter.

---

### Concatenation

```sql
-- Standard / MySQL / MariaDB:
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM employees;

-- PostgreSQL:
SELECT first_name || ' ' || last_name AS full_name FROM employees;

-- SQL Server:
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM employees;
-- or: first_name + ' ' + last_name
```

```ad-note
`CONCAT` in MySQL treats NULL as empty string. The `||` operator in PostgreSQL returns NULL if any operand is NULL. Use [[COALESCE and IFNULL]] to handle NULLs.
```

---

### Substring

```sql
-- Extract characters starting at position, for length:
SELECT SUBSTRING(name, 1, 3) FROM users;   -- first 3 characters
-- MySQL also supports: SUBSTR(name, 1, 3)
-- SQL Server: SUBSTRING(name, 1, 3)
```

---

### Length

```sql
SELECT LENGTH(name) FROM users;     -- MySQL, PostgreSQL
SELECT LEN(name) FROM users;        -- SQL Server (excludes trailing spaces)
```

---

### Case Conversion

```sql
SELECT UPPER(name) FROM users;   -- 'alice' → 'ALICE'
SELECT LOWER(name) FROM users;   -- 'ALICE' → 'alice'
```

---

### Trim

```sql
SELECT TRIM(name) FROM users;          -- remove leading and trailing whitespace
SELECT LTRIM(name) FROM users;         -- remove leading whitespace only
SELECT RTRIM(name) FROM users;         -- remove trailing whitespace only
SELECT TRIM('x' FROM name) FROM users; -- remove specific character (standard SQL)
```

---

### Replace

```sql
SELECT REPLACE(phone, '-', '') FROM users;  -- '555-1234' → '5551234'
```

---

### Left / Right

```sql
SELECT LEFT(name, 5) FROM users;    -- first 5 characters
SELECT RIGHT(name, 3) FROM users;   -- last 3 characters
```

---

### Find Substring Position

```sql
-- MySQL:
SELECT LOCATE('son', last_name) FROM employees;   -- returns position (1-based) or 0

-- PostgreSQL:
SELECT POSITION('son' IN last_name) FROM employees;

-- SQL Server:
SELECT CHARINDEX('son', last_name) FROM employees;
```

---

### Quick Reference

| Function                     | MySQL            | PostgreSQL       | SQL Server       |
| ---------------------------- | ---------------- | ---------------- | ---------------- |
| Concatenate                  | `CONCAT()`       | `\|\|`           | `CONCAT()` / `+` |
| Length                       | `LENGTH()`       | `LENGTH()`       | `LEN()`          |
| Substring                   | `SUBSTRING()`    | `SUBSTRING()`    | `SUBSTRING()`    |
| Find position               | `LOCATE()`       | `POSITION()`     | `CHARINDEX()`    |
| Pad                         | `LPAD()` / `RPAD()` | `LPAD()` / `RPAD()` | N/A (use `FORMAT`) |
