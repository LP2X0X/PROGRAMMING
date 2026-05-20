---
tags: [sql, functions, type-conversion]
---

- Type conversion (casting) changes a value from one data type to another. Needed when types don't match in comparisons, calculations, or output formatting.

---

### CAST (Standard SQL)

- Works in all major RDBMS:
```sql
SELECT CAST(price AS INT) FROM products;              -- decimal → integer
SELECT CAST(123 AS VARCHAR(10));                       -- number → string
SELECT CAST('2024-01-15' AS DATE);                     -- string → date
SELECT CAST(salary AS DECIMAL(10, 2)) FROM employees;  -- adjust precision
```

---

### CONVERT

```sql
-- MySQL syntax:
SELECT CONVERT(price, SIGNED);          -- to signed integer
SELECT CONVERT('2024-01-15', DATE);     -- string to date

-- SQL Server syntax (has a style parameter):
SELECT CONVERT(VARCHAR, hire_date, 23);  -- date to 'YYYY-MM-DD' string
SELECT CONVERT(INT, '123');              -- string to integer
```

---

### Implicit vs Explicit Conversion

- **Implicit**: the database converts automatically when types don't match:
```sql
SELECT '100' + 50;  -- SQL Server: 150 (string implicitly converted to int)
                     -- MySQL: 150
                     -- PostgreSQL: ERROR (no implicit conversion)
```

- **Explicit**: you specify the conversion with CAST/CONVERT.

```ad-tip
Always prefer **explicit** conversion. Implicit conversion can produce unexpected results and can prevent [[What are Indexes|index usage]] (e.g., comparing a VARCHAR column to an INT).
```

---

### Common Conversions

```sql
-- String to number:
SELECT CAST('42.5' AS DECIMAL(5, 1));

-- Number to string:
SELECT CAST(42 AS VARCHAR(10));

-- String to date:
SELECT CAST('2024-01-15' AS DATE);

-- Date to string (for display):
SELECT CAST(hire_date AS VARCHAR(10)) FROM employees;
-- But prefer DATE_FORMAT/TO_CHAR/FORMAT for control. See [[Date Functions]].
```

---

### TRY_CAST (SQL Server)

- Returns `NULL` instead of an error when conversion fails:
```sql
SELECT TRY_CAST('abc' AS INT);    -- NULL (instead of error)
SELECT TRY_CAST('123' AS INT);    -- 123
```

- MySQL and PostgreSQL don't have `TRY_CAST` — use `CASE` with validation or error handling instead.

---

### Pitfalls

- **Precision loss**: `CAST(3.99 AS INT)` → `3` (truncates, doesn't round).
- **Overflow**: `CAST(999999999999 AS INT)` may fail or produce wrong results.
- **Date format**: `CAST('01/15/2024' AS DATE)` may fail or be ambiguous. Use [[Date Functions]] with explicit format strings.
