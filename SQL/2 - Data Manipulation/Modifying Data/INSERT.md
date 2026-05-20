---
tags: [sql, dml, modifying-data]
---

- `INSERT` adds new rows to a table.

---

### Insert a Single Row

```sql
INSERT INTO employees (first_name, last_name, department, salary)
VALUES ('John', 'Doe', 'Engineering', 75000);
```

- Always specify the column list explicitly — it makes the statement resilient to schema changes and easier to read.

---

### Insert Multiple Rows

```sql
INSERT INTO employees (first_name, last_name, department, salary)
VALUES 
    ('Jane', 'Smith', 'Marketing', 68000),
    ('Bob', 'Wilson', 'Engineering', 82000),
    ('Alice', 'Brown', 'Sales', 71000);
```

- Much faster than separate `INSERT` statements — fewer round trips to the database.

---

### Insert from a Query

```sql
INSERT INTO archived_orders (order_id, customer_id, total)
SELECT order_id, customer_id, total
FROM orders
WHERE order_date < '2024-01-01';
```

- The column count and types of the `SELECT` must match the `INSERT` target.

---

### AUTO_INCREMENT / IDENTITY

- A column that automatically generates a unique value for each new row:
```sql
CREATE TABLE products (
    id INT AUTO_INCREMENT PRIMARY KEY,  -- MySQL / MariaDB
    name VARCHAR(100)
);
```
- SQL Server uses `IDENTITY(1,1)`, PostgreSQL uses `SERIAL` or `GENERATED ALWAYS AS IDENTITY`.
- You typically omit this column in `INSERT` — the database fills it automatically.

---

### DEFAULT Values

- If a column has a `DEFAULT` defined, omitting it from `INSERT` uses the default:
```sql
INSERT INTO orders (customer_id, status)
VALUES (42, DEFAULT);  -- status gets its default value
```

```ad-warning
Omitting the column list entirely (`INSERT INTO table VALUES (...)`) requires you to provide values for ALL columns in the exact table order. This is fragile — avoid it.
```
