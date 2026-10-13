---
tags: [sql, dml, reading-data]
---

- `SELECT` is the most frequently used SQL statement. It retrieves data from one or more tables.

```ad-note
The select clause determines which of all possible columns should be included in the query’s result set.
```

---

### Basic Syntax

```sql
SELECT column1, column2 FROM table_name;
```

- `SELECT *` retrieves all columns. Useful for exploration, but in production code always specify the columns you need — it's faster, clearer, and won't break if columns are added later.

---

### [[Column Alias|Column Aliases]]

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

```ad-warning
Generating a distinct set of results requires the data to be sorted, which can be time consuming for large [[Result Set|result sets]]. Don't use `DISTINCT` just to be safe — take the time to understand your data so you know whether duplicates are actually possible.
```

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

---

### Building a Table On the Fly

- A `SELECT` without `FROM` produces **one row**. Stack multiple rows with `UNION ALL` to create a table without any real table existing:

```sql
SELECT 'Small Fry' AS name, 0 AS low, 74.99 AS high
UNION ALL
SELECT 'Average Joes', 75, 149.99
UNION ALL
SELECT 'Heavy Hitters', 150, 9999999.99;
```

| name          | low | high       |
| ------------- | --- | ---------- |
| Small Fry     |   0 |      74.99 |
| Average Joes  |  75 |     149.99 |
| Heavy Hitters | 150 | 9999999.99 |

- Column names come from the **first** `SELECT` only — the rest just provide values in the same column order
- Useful inside CTEs or subqueries when you need lookup data that doesn't live in a table (e.g. custom grouping buckets, threshold ranges, label mappings):

```sql
WITH levels AS (
    SELECT 'Junior' AS level, 0 AS min_exp, 2 AS max_exp
    UNION ALL
    SELECT 'Mid', 3, 5
    UNION ALL
    SELECT 'Senior', 6, 99
)
SELECT e.name, levels.level
FROM employees e
INNER JOIN levels ON e.years_exp BETWEEN levels.min_exp AND levels.max_exp;
```

```ad-note
`SELECT` does not modify any data — it is a purely read-only operation. See [[INSERT]], [[UPDATE]], and [[DELETE]] for data modification.
```
