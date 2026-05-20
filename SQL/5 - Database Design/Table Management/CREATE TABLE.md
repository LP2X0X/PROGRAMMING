---
tags: [sql, ddl, table-management]
---

- `CREATE TABLE` defines a new table with its columns, data types, and constraints. This is a **DDL** (Data Definition Language) statement. See [[SQL Syntax Basics]] for DDL vs DML.

---

### Basic Syntax

```sql
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    department_id INT,
    salary DECIMAL(10, 2) DEFAULT 0.00,
    hire_date DATE NOT NULL,
    active BOOLEAN DEFAULT TRUE,
    FOREIGN KEY (department_id) REFERENCES departments(id)
);
```

---

### IF NOT EXISTS

- Prevents an error if the table already exists:
```sql
CREATE TABLE IF NOT EXISTS employees (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);
```

---

### CREATE TABLE AS SELECT (CTAS)

- Create a new table from the result of a query:
```sql
CREATE TABLE archived_orders AS
SELECT * FROM orders
WHERE order_date < '2023-01-01';
```

- The new table has the same column types as the query result.
- Constraints (primary key, foreign key, indexes) are **not** copied — only the data and column definitions.

---

### Temporary Tables

```sql
CREATE TEMPORARY TABLE temp_results (
    id INT,
    score DECIMAL(5, 2)
);
```

- Temporary tables are only visible to the current session and are automatically dropped when the session ends.
- Useful for storing intermediate results in complex queries or stored procedures.

---

### Inline vs Table-Level Constraints

```sql
-- Inline constraint (on the column):
CREATE TABLE users (
    id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE NOT NULL
);

-- Table-level constraint (after all columns):
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id),  -- composite primary key
    FOREIGN KEY (order_id) REFERENCES orders(id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);
```

- Composite [[Primary Key|primary keys]] and named constraints must use table-level syntax.
- See [[Foreign Key]], [[Unique, Not Null, Check, Default]] for constraint details.
