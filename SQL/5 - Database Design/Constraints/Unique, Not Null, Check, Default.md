---
tags: [sql, ddl, constraints]
---

- These are the most common **column constraints** used to enforce data integrity at the database level.

---

### NOT NULL

- The column **cannot** contain NULL values. Every row must have a value for this column:
```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    email VARCHAR(100) NOT NULL,
    name VARCHAR(50) NOT NULL
);
```

- Attempting to insert a NULL into a NOT NULL column causes an error.

---

### UNIQUE

- No two rows can have the same value in this column (but NULLs may be allowed depending on the RDBMS):
```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE NOT NULL,
    username VARCHAR(50) UNIQUE
);
```

- A UNIQUE constraint automatically creates an index on the column.
- **UNIQUE vs PRIMARY KEY**: a table can have only one primary key but multiple UNIQUE constraints. Primary key implies NOT NULL; UNIQUE does not (except in SQL Server where UNIQUE allows only one NULL).

---

### CHECK

- Validates that values meet a specified condition:
```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    age INT CHECK (age >= 18 AND age <= 120),
    salary DECIMAL(10, 2) CHECK (salary >= 0),
    status VARCHAR(10) CHECK (status IN ('active', 'inactive', 'pending'))
);
```

- The CHECK condition must evaluate to TRUE or NULL (NULL passes the check).

```ad-note
MySQL accepted but **ignored** CHECK constraints before version 8.0.16. If you're on an older MySQL, CHECK won't actually enforce anything — use triggers or application-level validation instead.
```

---

### DEFAULT

- Provides a value when none is specified during INSERT:
```sql
CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    quantity INT DEFAULT 1
);
```

```sql
INSERT INTO orders (id) VALUES (1);
-- status = 'pending', created_at = now, quantity = 1
```

---

### Combining Constraints

- Multiple constraints can be applied to a single column:
```sql
email VARCHAR(100) UNIQUE NOT NULL,
age INT NOT NULL CHECK (age >= 0) DEFAULT 0,
```

---

### Named Constraints

- Naming constraints makes them easier to reference in [[ALTER TABLE]] operations:
```sql
CREATE TABLE products (
    id INT PRIMARY KEY,
    price DECIMAL(10, 2),
    stock INT,
    CONSTRAINT chk_price_positive CHECK (price > 0),
    CONSTRAINT chk_stock_nonneg CHECK (stock >= 0)
);

-- Easy to drop later:
ALTER TABLE products DROP CONSTRAINT chk_price_positive;
```
