---
tags: [sql, functions, null-handling]
---

- Functions for handling `NULL` values — providing defaults, avoiding NULL propagation, and cleaning up output. See [[IS NULL]] for NULL fundamentals.

---

### COALESCE (Standard SQL — use this)

- Returns the **first non-NULL** value from a list of arguments:
```sql
SELECT COALESCE(phone, mobile, email, 'No contact') AS contact
FROM customers;
-- Tries phone first, then mobile, then email, finally falls back to a string
```

- Two-argument form (most common):
```sql
SELECT 
    name,
    COALESCE(nickname, name) AS display_name
FROM users;
```

```ad-tip
`COALESCE` is **standard SQL** and works in all RDBMS. Prefer it over the vendor-specific alternatives below.
```

---

### IFNULL (MySQL / MariaDB)

```sql
SELECT IFNULL(phone, 'N/A') FROM customers;
-- Returns phone if not NULL, otherwise 'N/A'
```

- Only takes **two** arguments (unlike COALESCE which takes many).

---

### ISNULL (SQL Server)

```sql
SELECT ISNULL(phone, 'N/A') FROM customers;
```

- Also only two arguments. Not to be confused with the `IS NULL` comparison operator.

---

### NVL (Oracle)

```sql
SELECT NVL(phone, 'N/A') FROM customers;
```

---

### NULLIF

- Returns `NULL` if the two arguments are **equal**, otherwise returns the first argument:
```sql
SELECT NULLIF(value, 0);
-- Returns NULL if value is 0, otherwise returns value
```

- Most common use: **avoid division by zero**:
```sql
SELECT total / NULLIF(count, 0) AS average
FROM stats;
-- If count is 0, NULLIF returns NULL, and total / NULL = NULL (not an error)
```

---

### Practical Patterns

```sql
-- Default value for missing data:
SELECT name, COALESCE(bio, 'No bio provided') AS bio FROM users;

-- Clean percentage display (avoid divide by zero):
SELECT 
    department,
    ROUND(100.0 * filled / NULLIF(total, 0), 1) AS fill_pct
FROM positions;

-- Concatenation that handles NULLs:
SELECT CONCAT(first_name, ' ', COALESCE(middle_name || ' ', ''), last_name)
FROM employees;
```
