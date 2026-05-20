---
tags: [sql, functions, numeric]
---

- Common numeric functions for rounding, absolute values, and mathematical operations.

---

### ROUND

```sql
SELECT ROUND(3.14159, 2);    -- 3.14
SELECT ROUND(3.145, 2);      -- 3.15 (rounds up at midpoint)
SELECT ROUND(1234.56, -2);   -- 1200 (negative decimals round to tens, hundreds, etc.)
```

---

### CEIL / CEILING and FLOOR

```sql
SELECT CEIL(4.1);     -- 5   (round up to nearest integer)
SELECT CEILING(4.1);  -- 5   (same, SQL Server uses CEILING)
SELECT FLOOR(4.9);    -- 4   (round down to nearest integer)

SELECT CEIL(-4.1);    -- -4  (toward positive infinity)
SELECT FLOOR(-4.9);   -- -5  (toward negative infinity)
```

---

### ABS

```sql
SELECT ABS(-42);    -- 42
SELECT ABS(42);     -- 42
```

---

### MOD (Modulo / Remainder)

```sql
SELECT MOD(10, 3);   -- 1   (MySQL, PostgreSQL)
SELECT 10 % 3;       -- 1   (all RDBMS)
```

---

### POWER and SQRT

```sql
SELECT POWER(2, 10);   -- 1024  (2^10)
SELECT SQRT(144);       -- 12
```

---

### GREATEST and LEAST

- Return the max/min from a **list of values** (not a column — that's `MAX`/`MIN`):
```sql
SELECT GREATEST(10, 20, 5);    -- 20
SELECT LEAST(10, 20, 5);       -- 5

-- Practical use: clamp a value to a range
SELECT LEAST(GREATEST(score, 0), 100) AS clamped_score
FROM results;  -- ensures score is between 0 and 100
```

```ad-note
`GREATEST` and `LEAST` are supported in MySQL and PostgreSQL but **not** in SQL Server. In SQL Server, use `IIF` or `CASE` expressions instead.
```

---

### Quick Reference

| Function          | Purpose                      | Example                |
| ----------------- | ---------------------------- | ---------------------- |
| `ROUND(n, d)`     | Round to `d` decimal places  | `ROUND(3.14159, 2)` → 3.14 |
| `CEIL(n)`         | Round up                     | `CEIL(4.1)` → 5       |
| `FLOOR(n)`        | Round down                   | `FLOOR(4.9)` → 4      |
| `ABS(n)`          | Absolute value               | `ABS(-5)` → 5         |
| `MOD(n, m)`       | Remainder                    | `MOD(10, 3)` → 1      |
| `POWER(n, exp)`   | Exponentiation               | `POWER(2, 3)` → 8     |
| `SQRT(n)`         | Square root                  | `SQRT(16)` → 4        |
