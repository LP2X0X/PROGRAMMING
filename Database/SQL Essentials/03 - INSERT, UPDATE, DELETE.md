---
tags: [sql, dml, data-modification]
---

- These three statements are how you **modify data** in a database. They are collectively called **DML** (Data Manipulation Language) along with `SELECT`.
  - `INSERT` — add new rows
  - `UPDATE` — modify existing rows
  - `DELETE` — remove rows
- Unlike `SELECT`, these operations **change your data permanently** (unless wrapped in a transaction). Treat them with respect.

---

## INSERT — Add New Rows

### Basic Syntax

```sql
INSERT INTO table_name (column1, column2, column3)
VALUES ('value1', 'value2', 'value3');
```

```sql
INSERT INTO employees (first_name, last_name, department, salary)
VALUES ('Alice', 'Chen', 'Engineering', 75000);
```

- Always specify the column list explicitly. Although `INSERT INTO employees VALUES (...)` works if you provide values for every column in the right order, it breaks silently if anyone adds or reorders columns.

### Identity / Auto-Increment Columns

- Most tables have a primary key column that auto-generates (identity in SQL Server, auto_increment in MySQL). **Do not** include it in your `INSERT`:

```sql
-- id is an identity column — don't specify it
INSERT INTO employees (first_name, last_name, department, salary)
VALUES ('Bob', 'Smith', 'Marketing', 65000);
-- The database assigns the next id automatically
```

- To find the generated id:

```sql
-- SQL Server
SELECT SCOPE_IDENTITY();

-- MySQL/MariaDB
SELECT LAST_INSERT_ID();
```

### Insert Multiple Rows

- Insert several rows in a single statement — much faster than multiple single-row inserts because it's one round trip to the database:

```sql
INSERT INTO employees (first_name, last_name, department, salary)
VALUES 
    ('Alice', 'Chen', 'Engineering', 75000),
    ('Bob', 'Smith', 'Marketing', 65000),
    ('Carol', 'Davis', 'Sales', 60000);
```

```ad-note
SQL Server limits multi-row `VALUES` to **1,000 rows** per statement. MySQL/MariaDB has no hard row limit but has a `max_allowed_packet` size limit (default 16MB). For bulk loading thousands of rows, use `BULK INSERT` (SQL Server) or `LOAD DATA INFILE` (MySQL).
```

### INSERT INTO ... SELECT

- Insert rows from the result of another query. Extremely powerful for data migration, archiving, or copying data between tables:

```sql
-- Archive all inactive users into a separate table
INSERT INTO archived_employees (first_name, last_name, department, salary)
SELECT first_name, last_name, department, salary
FROM employees
WHERE active = 0;
```

- The column count and types must match between the `INSERT` column list and the `SELECT` output.

### INSERT with Default Values

```sql
-- If columns have defaults, you can omit them
INSERT INTO employees (first_name, last_name)
VALUES ('Dave', 'Wilson');
-- department defaults to NULL (or whatever DEFAULT is defined)

-- Insert a row with ALL defaults
INSERT INTO log_entries DEFAULT VALUES;  -- SQL Server
INSERT INTO log_entries () VALUES ();     -- MySQL
```

---

## UPDATE — Modify Existing Rows

### Basic Syntax

```sql
UPDATE table_name
SET column1 = new_value1,
    column2 = new_value2
WHERE condition;
```

```sql
-- Give Alice a raise
UPDATE employees
SET salary = 80000
WHERE first_name = 'Alice' AND last_name = 'Chen';
```

### Update Multiple Columns

```sql
UPDATE employees
SET salary = 85000,
    department = 'Senior Engineering',
    updated_at = GETDATE()   -- SQL Server
WHERE employee_id = 42;
```

### Update with Expressions

```sql
-- Give everyone in Engineering a 10% raise
UPDATE employees
SET salary = salary * 1.10
WHERE department = 'Engineering';

-- Set bonus to 5% of salary for all active employees
UPDATE employees
SET bonus = salary * 0.05
WHERE active = 1;
```

### The Golden Rule of UPDATE

```ad-warning
**NEVER forget the `WHERE` clause.** An `UPDATE` without `WHERE` modifies **every single row** in the table.

```sql
-- DISASTER: every user in the entire table is now an admin
UPDATE users SET is_admin = 1;

-- What you meant:
UPDATE users SET is_admin = 1 WHERE user_id = 42;
```

Before running any `UPDATE` in production, first run the same query as a `SELECT` to verify which rows will be affected:

```sql
-- Step 1: Preview what will be changed
SELECT * FROM users WHERE user_id = 42;

-- Step 2: If it looks right, do the update
UPDATE users SET is_admin = 1 WHERE user_id = 42;
```
```

### UPDATE with JOIN

- Sometimes you need to update one table based on data in another. The syntax differs between engines.

**SQL Server:**

```sql
UPDATE e
SET e.department_name = d.name
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
```

**MySQL/MariaDB:**

```sql
UPDATE employees e
INNER JOIN departments d ON e.department_id = d.id
SET e.department_name = d.name;
```

---

## DELETE — Remove Rows

### Basic Syntax

```sql
DELETE FROM table_name
WHERE condition;
```

```sql
-- Remove a specific employee
DELETE FROM employees
WHERE employee_id = 42;

-- Remove all inactive users
DELETE FROM employees
WHERE active = 0;
```

### The Same Golden Rule

```ad-warning
**`DELETE` without `WHERE` deletes ALL rows.** The table still exists but it's empty.

```sql
-- DISASTER: every row gone
DELETE FROM employees;

-- What you meant:
DELETE FROM employees WHERE employee_id = 42;
```

Same technique — preview first:

```sql
-- Step 1: What will be deleted?
SELECT * FROM employees WHERE last_login < '2020-01-01';

-- Step 2: If safe, delete
DELETE FROM employees WHERE last_login < '2020-01-01';
```
```

### DELETE with JOIN

- Delete rows from one table based on data in another.

**SQL Server:**

```sql
DELETE e
FROM employees e
INNER JOIN departments d ON e.department_id = d.id
WHERE d.name = 'Deprecated';
```

**MySQL/MariaDB:**

```sql
DELETE e
FROM employees e
INNER JOIN departments d ON e.department_id = d.id
WHERE d.name = 'Deprecated';
```

---

## TRUNCATE TABLE vs DELETE

| Feature | `DELETE FROM table` | `TRUNCATE TABLE table` |
| --- | --- | --- |
| Removes | Rows matching `WHERE` (or all if no `WHERE`) | **All** rows — no `WHERE` allowed |
| Logging | Logs each deleted row (slower) | Minimal logging — deallocates pages (faster) |
| Identity reset | No — next insert continues from last value | Yes — resets auto-increment/identity to seed |
| Triggers | Fires `DELETE` triggers | Does **not** fire triggers |
| Rollback | Can be rolled back in a transaction | SQL Server: can be rolled back. MySQL: **cannot** be rolled back (implicit commit) |
| Foreign keys | Works if FK constraints allow | Fails if table is referenced by a foreign key |
| Speed | Slower for large tables | Much faster |

```sql
-- Remove all rows but keep the table structure
TRUNCATE TABLE temp_data;
```

```ad-important
Use `TRUNCATE` when you want to quickly empty an entire table (e.g., staging tables, temp data). Use `DELETE` when you need `WHERE` conditions, need triggers to fire, or need guaranteed rollback support.
```

---

## Transaction Safety — Preview Before Committing

- For any destructive operation, wrap it in a transaction so you can review before making it permanent:

**SQL Server:**

```sql
BEGIN TRANSACTION;

-- Do the dangerous thing
DELETE FROM employees WHERE department = 'Obsolete';

-- Check: how many rows were affected?
-- Look at the "rows affected" message

-- If it looks wrong:
ROLLBACK;

-- If it looks right:
COMMIT;
```

**MySQL/MariaDB:**

```sql
START TRANSACTION;

DELETE FROM employees WHERE department = 'Obsolete';

-- Review
SELECT COUNT(*) FROM employees WHERE department = 'Obsolete';
-- Should be 0 if the delete worked

-- Undo if wrong:
ROLLBACK;

-- Commit if right:
COMMIT;
```

```ad-note
By default, MySQL/MariaDB runs in **autocommit** mode — each statement is automatically committed. To use transactions, either `START TRANSACTION` explicitly or set `SET autocommit = 0`. SQL Server also defaults to autocommit but `BEGIN TRANSACTION` overrides it for the scope of that transaction.
```

---

## OUTPUT / RETURNING — Get Back What You Changed

- After an `INSERT`, `UPDATE`, or `DELETE`, you often want to see exactly what rows were affected.

**SQL Server — `OUTPUT` clause:**

```sql
-- See what was inserted
INSERT INTO employees (first_name, last_name, salary)
OUTPUT INSERTED.employee_id, INSERTED.first_name, INSERTED.salary
VALUES ('Eve', 'Johnson', 70000);

-- See what was deleted
DELETE FROM employees
OUTPUT DELETED.employee_id, DELETED.first_name
WHERE department = 'Obsolete';

-- See before and after for updates
UPDATE employees
SET salary = salary * 1.10
OUTPUT DELETED.salary AS old_salary, INSERTED.salary AS new_salary, INSERTED.first_name
WHERE department = 'Engineering';
```

**MySQL/MariaDB:**

- MySQL does not have a `RETURNING` clause (PostgreSQL does). The workaround is `LAST_INSERT_ID()` for inserts or `SELECT` + `UPDATE` in a transaction:

```sql
-- Insert and get the new id
INSERT INTO employees (first_name, last_name, salary)
VALUES ('Eve', 'Johnson', 70000);
SELECT LAST_INSERT_ID();

-- MariaDB 10.5+ supports RETURNING for INSERT and DELETE:
INSERT INTO employees (first_name, last_name, salary)
VALUES ('Eve', 'Johnson', 70000)
RETURNING employee_id, first_name;

DELETE FROM employees
WHERE department = 'Obsolete'
RETURNING employee_id, first_name;
```

---

## Common Patterns

### Upsert — Insert or Update

- Insert a row, but if it already exists (by primary key or unique constraint), update it instead.

**SQL Server — `MERGE`:**

```sql
MERGE INTO employees AS target
USING (SELECT 42 AS id, 'Alice' AS first_name, 80000 AS salary) AS source
ON target.employee_id = source.id
WHEN MATCHED THEN
    UPDATE SET salary = source.salary
WHEN NOT MATCHED THEN
    INSERT (first_name, salary) VALUES (source.first_name, source.salary);
```

**MySQL/MariaDB — `ON DUPLICATE KEY UPDATE`:**

```sql
INSERT INTO employees (employee_id, first_name, salary)
VALUES (42, 'Alice', 80000)
ON DUPLICATE KEY UPDATE salary = VALUES(salary);
```

### Soft Delete

- Instead of actually deleting rows, mark them as deleted. This preserves data for auditing and undo:

```sql
-- "Delete" by marking
UPDATE employees
SET is_deleted = 1, deleted_at = GETDATE()
WHERE employee_id = 42;

-- All queries must filter: WHERE is_deleted = 0
SELECT * FROM employees WHERE is_deleted = 0;
```

```ad-note
Now that you can read, filter, and modify data, the next step is combining data from multiple tables: [[04 - JOIN]].
```
