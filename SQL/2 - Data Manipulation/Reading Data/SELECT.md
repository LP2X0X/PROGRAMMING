---
tags: [sql, dml, reading-data]
---

- `SELECT` is the most frequently used SQL statement. It retrieves data from one or more tables.

---

### Basic Syntax

```sql
SELECT column1, column2 FROM table_name;
```

- `SELECT *` retrieves all columns. Useful for exploration, but in production code always specify the columns you need — it's faster, clearer, and won't break if columns are added later.

---

### Column Aliases

- Use `AS` to rename a column in the result:
```sql
SELECT first_name AS name, salary * 12 AS annual_salary
FROM employees;
```
- The `AS` keyword is optional — `SELECT first_name name` works too, but `AS` is clearer.

---

### DISTINCT

- Removes duplicate rows from the result:
```sql
SELECT DISTINCT department FROM employees;
```
- `DISTINCT` applies to the **entire row**, not just one column. `SELECT DISTINCT a, b` returns unique combinations of `a` and `b`.

---

### Expressions in SELECT

- You can use arithmetic, functions, and string operations directly in `SELECT`:
```sql
SELECT 
    product_name,
    price,
    price * 0.9 AS discounted_price,
    UPPER(product_name) AS upper_name
FROM products;
```

---

### SELECT Without FROM

- Some RDBMS allow `SELECT` without a table for quick calculations:
```sql
SELECT 1 + 1;           -- Returns 2
SELECT NOW();            -- Returns current timestamp
SELECT 'hello' AS greet; -- Returns a string
```
- In Oracle, you must use `FROM dual` for this pattern.

```ad-note
`SELECT` does not modify any data — it is a purely read-only operation. See [[INSERT]], [[UPDATE]], and [[DELETE]] for data modification.
```
