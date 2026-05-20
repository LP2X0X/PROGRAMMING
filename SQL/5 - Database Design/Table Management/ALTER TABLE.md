---
tags: [sql, ddl, table-management]
---

- `ALTER TABLE` modifies an existing table's structure — add/drop columns, change types, add/remove constraints.

---

### Add a Column

```sql
ALTER TABLE employees
ADD COLUMN phone VARCHAR(20);

-- With a default value:
ALTER TABLE employees
ADD COLUMN status VARCHAR(10) DEFAULT 'active' NOT NULL;
```

---

### Drop a Column

```sql
ALTER TABLE employees
DROP COLUMN phone;
```

---

### Modify a Column

```sql
-- MySQL / MariaDB:
ALTER TABLE employees
MODIFY COLUMN salary DECIMAL(12, 2) NOT NULL;

-- PostgreSQL:
ALTER TABLE employees
ALTER COLUMN salary TYPE DECIMAL(12, 2);

ALTER TABLE employees
ALTER COLUMN salary SET NOT NULL;

-- SQL Server:
ALTER TABLE employees
ALTER COLUMN salary DECIMAL(12, 2) NOT NULL;
```

---

### Rename a Column

```sql
-- MySQL 8+ / PostgreSQL:
ALTER TABLE employees
RENAME COLUMN last_name TO surname;
```

---

### Rename a Table

```sql
-- MySQL:
RENAME TABLE employees TO staff;

-- PostgreSQL / SQL Server:
ALTER TABLE employees RENAME TO staff;
```

---

### Add / Drop Constraints

```sql
-- Add a unique constraint:
ALTER TABLE employees
ADD CONSTRAINT uq_email UNIQUE (email);

-- Add a foreign key:
ALTER TABLE orders
ADD CONSTRAINT fk_customer
FOREIGN KEY (customer_id) REFERENCES customers(id);

-- Drop a constraint:
ALTER TABLE employees
DROP CONSTRAINT uq_email;

-- Drop a foreign key (MySQL):
ALTER TABLE orders
DROP FOREIGN KEY fk_customer;
```

---

### Add / Drop Index

```sql
-- Add:
CREATE INDEX idx_last_name ON employees(last_name);

-- Drop (MySQL):
DROP INDEX idx_last_name ON employees;

-- Drop (PostgreSQL / SQL Server):
DROP INDEX idx_last_name;
```

```ad-warning
Some `ALTER TABLE` operations lock the table for the duration of the change (especially in MySQL). On large production tables, adding a column or index can block reads/writes for minutes. Look into online DDL options (`ALGORITHM=INPLACE` in MySQL, `CONCURRENTLY` in PostgreSQL) for non-blocking changes. See [[What are Indexes]].
```
