---
tags: [sql, views, programmability]
---

- **Views**, **stored procedures**, and **functions** are database objects that save and reuse SQL logic. They are the building blocks of database-level abstraction — letting you define complex queries once and reference them by name.
- Views act as virtual tables. Stored procedures are saved programs. Functions return computed values.

---

## Views — Saved Queries as Virtual Tables

- A **view** is a named `SELECT` query stored in the database. When you query the view, the database runs the underlying query and returns the result — as if it were a table.

### Creating a View

```sql
CREATE VIEW ActiveUsers AS
SELECT id, first_name, last_name, email
FROM users
WHERE is_active = 1;
```

- Now use it like any table:

```sql
SELECT * FROM ActiveUsers;

SELECT first_name, email 
FROM ActiveUsers 
WHERE last_name LIKE 'S%';
```

### Key Characteristics

- **Not a copy of data** — the view re-executes the underlying query each time it's accessed. If the base table changes, the view reflects those changes instantly.
- **No storage cost** — a view stores only the query definition, not the data.
- **Can be JOINed, filtered, aggregated** — anything you can do with a table, you can do with a view.

### Why Use Views?

| Benefit | Example |
| --- | --- |
| **Simplify complex queries** | Wrap a 10-table JOIN in a view, then `SELECT * FROM SalesSummary` |
| **Abstraction layer** | If the underlying schema changes, update the view — consumers don't need to change |
| **Security** | Grant users access to the view (limited columns) without granting access to the base table |
| **Reusability** | Define the logic once, use it in many queries across applications |
| **Consistent business logic** | "Active user" means `WHERE is_active = 1 AND deleted_at IS NULL` — define it once in a view |

### CREATE OR REPLACE

- Modify an existing view without dropping and recreating it:

```sql
-- MySQL/MariaDB
CREATE OR REPLACE VIEW ActiveUsers AS
SELECT id, first_name, last_name, email, created_at
FROM users
WHERE is_active = 1;
```

```sql
-- SQL Server (uses ALTER)
ALTER VIEW ActiveUsers AS
SELECT id, first_name, last_name, email, created_at
FROM users
WHERE is_active = 1;
```

### Dropping a View

```sql
DROP VIEW ActiveUsers;
DROP VIEW IF EXISTS ActiveUsers;  -- avoids error if it doesn't exist
```

### Updatable Views

- Simple views (single table, no aggregates, no `DISTINCT`, no `GROUP BY`, no `UNION`) can support `INSERT`, `UPDATE`, and `DELETE`:

```sql
-- This modifies the underlying users table
UPDATE ActiveUsers SET email = 'new@mail.com' WHERE id = 42;

-- This inserts into the users table
INSERT INTO ActiveUsers (first_name, last_name, email)
VALUES ('Dave', 'Wilson', 'dave@mail.com');
```

- Complex views (with JOINs, aggregates, subqueries, or `DISTINCT`) are generally **read-only**.

### WITH CHECK OPTION

- Prevents inserts/updates through a view that would make the row disappear from the view:

```sql
CREATE VIEW ActiveUsers AS
SELECT id, first_name, last_name, email
FROM users
WHERE is_active = 1
WITH CHECK OPTION;

-- This would FAIL: setting is_active = 0 makes the row invisible to the view
UPDATE ActiveUsers SET is_active = 0 WHERE id = 42;
```

### Views and Performance

```ad-warning
Regular views do **not** improve query performance. The underlying query runs every time you access the view. Don't create a view expecting it to be "cached."

If you need pre-computed results for performance, look into:
- **Indexed views** (SQL Server) — materializes the view result and keeps it updated
- **Materialized views** (PostgreSQL, Oracle) — stores the result on disk, manually refreshed
- MySQL/MariaDB does **not** support materialized views — use a table + scheduled refresh instead
```

---

## Stored Procedures — Saved SQL Programs

- A **stored procedure** is a named block of SQL statements saved in the database. It can accept parameters, contain control flow logic (IF/ELSE, loops), modify data, and return result sets.
- Think of it as a function in your application code, but it lives in the database.

### Creating a Stored Procedure

**SQL Server:**

```sql
CREATE PROCEDURE sp_GetUserOrders
    @UserId INT
AS
BEGIN
    SELECT o.order_id, o.total, o.order_date, p.product_name
    FROM orders o
    INNER JOIN order_items oi ON o.order_id = oi.order_id
    INNER JOIN products p ON oi.product_id = p.product_id
    WHERE o.user_id = @UserId
    ORDER BY o.order_date DESC;
END;
```

- Call it:

```sql
EXEC sp_GetUserOrders @UserId = 42;
-- or
EXEC sp_GetUserOrders 42;
```

**MySQL/MariaDB:**

```sql
DELIMITER //
CREATE PROCEDURE sp_GetUserOrders(IN p_UserId INT)
BEGIN
    SELECT o.order_id, o.total, o.order_date, p.product_name
    FROM orders o
    INNER JOIN order_items oi ON o.order_id = oi.order_id
    INNER JOIN products p ON oi.product_id = p.product_id
    WHERE o.user_id = p_UserId
    ORDER BY o.order_date DESC;
END //
DELIMITER ;
```

- Call it:

```sql
CALL sp_GetUserOrders(42);
```

```ad-note
MySQL requires `DELIMITER //` because the procedure body contains semicolons, which MySQL would otherwise interpret as the end of the `CREATE` statement. SQL Server doesn't need this. The `//` is restored to `;` after the procedure with `DELIMITER ;`.
```

### Parameters

**SQL Server parameters:**

```sql
CREATE PROCEDURE sp_UpdateSalary
    @EmployeeId INT,          -- required input
    @NewSalary DECIMAL(10,2), -- required input
    @OldSalary DECIMAL(10,2) OUTPUT  -- output parameter
AS
BEGIN
    SELECT @OldSalary = salary FROM employees WHERE id = @EmployeeId;
    UPDATE employees SET salary = @NewSalary WHERE id = @EmployeeId;
END;

-- Usage:
DECLARE @PreviousSalary DECIMAL(10,2);
EXEC sp_UpdateSalary @EmployeeId = 42, @NewSalary = 85000, @OldSalary = @PreviousSalary OUTPUT;
SELECT @PreviousSalary;
```

**MySQL/MariaDB parameters:**

| Type | Direction | Usage |
| --- | --- | --- |
| `IN` | Input only (default) | Pass values into the procedure |
| `OUT` | Output only | Return values to the caller |
| `INOUT` | Both directions | Pass in and get modified value back |

```sql
DELIMITER //
CREATE PROCEDURE sp_CountByDept(IN p_dept VARCHAR(50), OUT p_count INT)
BEGIN
    SELECT COUNT(*) INTO p_count FROM employees WHERE department = p_dept;
END //
DELIMITER ;

-- Usage:
CALL sp_CountByDept('Engineering', @result);
SELECT @result;
```

### Control Flow in Stored Procedures

**SQL Server:**

```sql
CREATE PROCEDURE sp_GiveRaise
    @EmployeeId INT,
    @Percentage DECIMAL(5,2)
AS
BEGIN
    DECLARE @CurrentSalary DECIMAL(10,2);
    DECLARE @MaxSalary DECIMAL(10,2) = 200000;

    SELECT @CurrentSalary = salary FROM employees WHERE id = @EmployeeId;

    IF @CurrentSalary IS NULL
    BEGIN
        RAISERROR('Employee not found', 16, 1);
        RETURN;
    END

    IF @CurrentSalary * (1 + @Percentage / 100) > @MaxSalary
    BEGIN
        PRINT 'Raise would exceed salary cap. Capping at max.';
        UPDATE employees SET salary = @MaxSalary WHERE id = @EmployeeId;
    END
    ELSE
    BEGIN
        UPDATE employees 
        SET salary = salary * (1 + @Percentage / 100) 
        WHERE id = @EmployeeId;
    END
END;
```

**MySQL/MariaDB:**

```sql
DELIMITER //
CREATE PROCEDURE sp_GiveRaise(IN p_emp_id INT, IN p_pct DECIMAL(5,2))
BEGIN
    DECLARE v_salary DECIMAL(10,2);
    DECLARE v_max DECIMAL(10,2) DEFAULT 200000;

    SELECT salary INTO v_salary FROM employees WHERE id = p_emp_id;

    IF v_salary IS NULL THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Employee not found';
    ELSEIF v_salary * (1 + p_pct / 100) > v_max THEN
        UPDATE employees SET salary = v_max WHERE id = p_emp_id;
    ELSE
        UPDATE employees 
        SET salary = salary * (1 + p_pct / 100) 
        WHERE id = p_emp_id;
    END IF;
END //
DELIMITER ;
```

### Modifying a Stored Procedure

```sql
-- SQL Server: ALTER to modify in place
ALTER PROCEDURE sp_GetUserOrders
    @UserId INT,
    @MinTotal DECIMAL(10,2) = 0  -- add a new optional parameter
AS
BEGIN
    SELECT o.order_id, o.total, o.order_date
    FROM orders o
    WHERE o.user_id = @UserId AND o.total >= @MinTotal
    ORDER BY o.order_date DESC;
END;

-- MySQL: must drop and recreate
DROP PROCEDURE IF EXISTS sp_GetUserOrders;
-- then CREATE PROCEDURE again
```

### Benefits and Drawbacks

| Pros | Cons |
| --- | --- |
| Reduce network round-trips (one call instead of many queries) | Hard to version control (lives in the database, not in source files) |
| Precompiled execution plan (can be faster) | Vendor-specific syntax — not portable |
| Encapsulate business logic at the database level | Can become hard to debug and unit test |
| Security — grant EXECUTE without exposing table access | Can hide complexity — "magic" black boxes |
| Reusable across multiple applications | Tight coupling to the database schema |

```ad-note
Modern development increasingly favors keeping business logic in application code (easier to test, version control, deploy, and review). Stored procedures are still valuable for **performance-critical batch operations**, **security-sensitive data access**, and **database-level auditing or enforcement**. Your existing SQL folder has more detail: [[Stored Procedures]].
```

---

## Functions — Return a Value

- A **function** is similar to a stored procedure but is designed to **return a value** and can be used inside `SELECT`, `WHERE`, and other SQL expressions.

### Scalar Function — Returns One Value

**SQL Server:**

```sql
CREATE FUNCTION fn_GetFullName(
    @FirstName VARCHAR(50), 
    @LastName VARCHAR(50)
)
RETURNS VARCHAR(101)
AS
BEGIN
    RETURN @FirstName + ' ' + @LastName;
END;

-- Usage: call with schema prefix
SELECT dbo.fn_GetFullName(first_name, last_name) AS full_name
FROM employees;

-- Use in WHERE
SELECT * FROM employees
WHERE dbo.fn_GetFullName(first_name, last_name) = 'Alice Chen';
```

**MySQL/MariaDB:**

```sql
DELIMITER //
CREATE FUNCTION fn_GetFullName(p_first VARCHAR(50), p_last VARCHAR(50))
RETURNS VARCHAR(101)
DETERMINISTIC
BEGIN
    RETURN CONCAT(p_first, ' ', p_last);
END //
DELIMITER ;

-- Usage: no schema prefix needed
SELECT fn_GetFullName(first_name, last_name) AS full_name
FROM employees;
```

```ad-note
MySQL requires you to declare functions as `DETERMINISTIC` (same input always returns same output) or `NOT DETERMINISTIC`. SQL Server doesn't require this declaration.
```

### Table-Valued Function (SQL Server Only)

- Returns an entire result set, not just a single value. You can `SELECT` from it like a table:

```sql
CREATE FUNCTION fn_GetOrdersByUser(@UserId INT)
RETURNS TABLE
AS
RETURN (
    SELECT order_id, total, order_date
    FROM orders
    WHERE user_id = @UserId
);

-- Usage: in FROM clause
SELECT * FROM dbo.fn_GetOrdersByUser(42);

-- Can be joined
SELECT u.first_name, f.total
FROM users u
CROSS APPLY dbo.fn_GetOrdersByUser(u.id) f;
```

```ad-warning
**Functions in WHERE clauses prevent index usage.** Calling a function on a column (e.g., `WHERE dbo.fn_GetFullName(first_name, last_name) = 'Alice Chen'`) means the function runs on every row — a full table scan. Use computed columns or indexed views instead if you need to filter on a function result frequently.
```

---

## View vs Stored Procedure vs Function

| Feature | View | Stored Procedure | Function |
| --- | --- | --- | --- |
| **What it stores** | A SELECT query | A block of SQL statements | Logic that returns a value |
| **Returns** | A virtual table (result set) | Result sets, output params, or nothing | A single value or table |
| **Usable in SELECT** | Yes (like a table) | No | Yes (in expressions) |
| **Usable in FROM** | Yes | No | Yes (table-valued, SQL Server) |
| **Can modify data** | Limited (simple updatable views) | Yes | No (SQL Server), Yes (MySQL) |
| **Accepts parameters** | No | Yes | Yes |
| **Control flow (IF/ELSE)** | No | Yes | Yes |
| **Precompiled plan** | No | Yes | Depends |
| **Primary use** | Simplify reads, security, abstraction | Batch operations, business logic | Computed values, reusable calculations |

---

## Schema Binding (SQL Server)

- `SCHEMABINDING` locks a view or function to the underlying table structure. You can't drop or alter the base table columns while the view depends on them:

```sql
CREATE VIEW ActiveUsers
WITH SCHEMABINDING
AS
SELECT id, first_name, last_name, email
FROM dbo.users
WHERE is_active = 1;
```

- Required for creating indexed (materialized) views in SQL Server.
- Prevents accidental breaking changes to the base tables.

---

## Practical Example — Tying It All Together

Here's a realistic scenario using all three:

```sql
-- 1. Function: compute order total with tax
CREATE FUNCTION fn_OrderTotalWithTax(@OrderId INT)
RETURNS DECIMAL(10,2)
AS
BEGIN
    DECLARE @total DECIMAL(10,2);
    SELECT @total = SUM(quantity * unit_price * 1.08)
    FROM order_items
    WHERE order_id = @OrderId;
    RETURN ISNULL(@total, 0);
END;

-- 2. View: summarize customer orders
CREATE VIEW CustomerOrderSummary AS
SELECT 
    c.id AS customer_id,
    c.name AS customer_name,
    COUNT(o.order_id) AS total_orders,
    SUM(o.total) AS total_spent,
    MAX(o.order_date) AS last_order_date
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.name;

-- 3. Stored procedure: generate a report for a date range
CREATE PROCEDURE sp_SalesReport
    @StartDate DATE,
    @EndDate DATE
AS
BEGIN
    SELECT 
        cos.customer_name,
        cos.total_orders,
        cos.total_spent,
        cos.last_order_date
    FROM CustomerOrderSummary cos
    WHERE cos.last_order_date BETWEEN @StartDate AND @EndDate
    ORDER BY cos.total_spent DESC;
END;

-- Use it:
EXEC sp_SalesReport @StartDate = '2026-01-01', @EndDate = '2026-06-30';
```

---

## What Comes Next

- This completes the SQL Essentials learning sequence. You now have the tools to read, filter, modify, join, aggregate, nest, and encapsulate SQL logic.
- For database **design** — normalization, indexes, constraints — see the [[Design]] folder.
- For advanced query techniques — window functions, set operations — see the SQL folder's [[4 - Advanced Queries]].
- For transactions and ACID properties — see [[Performance and Administration]].
