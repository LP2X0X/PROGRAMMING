---
tags: [sql, dml, modifying-data]
---

- `UPDATE` modifies existing rows in a table.

---

### Basic Syntax

```sql
UPDATE employees
SET salary = 80000
WHERE employee_id = 42;
```

---

### Update Multiple Columns

```sql
UPDATE employees
SET salary = 85000,
    department = 'Senior Engineering',
    updated_at = NOW()
WHERE employee_id = 42;
```

---

### UPDATE Without WHERE

```ad-warning
An `UPDATE` without a `WHERE` clause modifies **every row** in the table. This is almost never what you want. Always double-check your `WHERE` clause before running an `UPDATE`.
```

```sql
-- DANGER: sets EVERY employee's salary to 0
UPDATE employees SET salary = 0;
```

- Best practice: run a `SELECT` with the same `WHERE` first to verify which rows will be affected.

---

### UPDATE with JOIN

- Update rows based on data from another table:
```sql
-- MySQL syntax
UPDATE orders o
JOIN customers c ON o.customer_id = c.id
SET o.discount = 0.10
WHERE c.membership = 'gold';
```

```sql
-- PostgreSQL / SQL Server syntax
UPDATE orders
SET discount = 0.10
FROM customers
WHERE orders.customer_id = customers.id
  AND customers.membership = 'gold';
```

---

### UPDATE with Subquery

```sql
UPDATE employees
SET salary = salary * 1.10
WHERE department_id = (
    SELECT id FROM departments WHERE name = 'Engineering'
);
```

```ad-tip
For large updates, consider doing them in batches with `LIMIT` (MySQL) to avoid locking the table for too long. See [[ACID Properties]] and [[BEGIN, COMMIT, ROLLBACK]] for transaction safety.
```
