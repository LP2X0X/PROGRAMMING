---
tags: [sql, ddl, constraints]
---

- A **primary key** uniquely identifies each row in a table. It enforces two constraints: **NOT NULL** and **UNIQUE**.

---

### Syntax

```sql
-- Inline:
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

-- With AUTO_INCREMENT (MySQL / MariaDB):
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100)
);

-- Table-level (required for composite keys):
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);
```

---

### Composite Primary Key

- A primary key consisting of **multiple columns**. The combination must be unique, not each column individually.
- Common in junction tables for many-to-many relationships. See [[Relational Databases]].

---

### Natural Key vs Surrogate Key

| Type           | Description                              | Example                        |
| -------------- | ---------------------------------------- | ------------------------------ |
| **Natural key** | A real-world attribute that is naturally unique | email, ISBN, SSN             |
| **Surrogate key** | An artificial identifier with no business meaning | auto-increment `id`, UUID  |

- **Surrogate keys are preferred** in most cases:
  - Stable — natural keys can change (email, name).
  - Compact — integer keys are small and fast for JOINs.
  - Simple — no multi-column complexity.
- Natural keys make sense when the value is truly immutable and widely used (e.g., ISO country codes).

---

### Rules

- Every table **should** have a primary key. Without one, you cannot uniquely reference a row.
- A table can have only **one** primary key (but it can span multiple columns).
- The database automatically creates an **index** on the primary key. In MySQL/InnoDB, this is the [[Clustered vs Non-Clustered Indexes|clustered index]].

```ad-note
Primary keys are the foundation that [[Foreign Key|foreign keys]] reference. A well-chosen primary key makes JOINs efficient and data integrity easy to maintain.
```
