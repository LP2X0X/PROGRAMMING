---
tags: [sql, ddl, constraints]
---

- A **foreign key** is a column (or set of columns) that references the [[Primary Key]] (or a UNIQUE key) of another table. It enforces **referential integrity** — you cannot insert a value that doesn't exist in the referenced table.

---

### Syntax

```sql
CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    total DECIMAL(10, 2),
    FOREIGN KEY (customer_id) REFERENCES customers(id)
);
```

- Named constraint (recommended for easier management):
```sql
CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT NOT NULL,
    CONSTRAINT fk_orders_customer 
        FOREIGN KEY (customer_id) REFERENCES customers(id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
);
```

---

### Referential Actions

- What happens when the referenced row (parent) is deleted or updated:

| Action        | ON DELETE behavior                              | ON UPDATE behavior                    |
| ------------- | ----------------------------------------------- | ------------------------------------- |
| `RESTRICT`    | Block the delete (default in most RDBMS)        | Block the update                      |
| `CASCADE`     | Delete all child rows too                       | Update the FK in all child rows       |
| `SET NULL`    | Set the FK column to NULL                       | Set the FK column to NULL             |
| `SET DEFAULT` | Set the FK column to its default value          | Set the FK column to its default      |
| `NO ACTION`   | Same as RESTRICT (checked at end of statement)  | Same as RESTRICT                      |

```sql
FOREIGN KEY (customer_id) REFERENCES customers(id)
    ON DELETE CASCADE      -- delete customer → delete all their orders
    ON UPDATE CASCADE      -- change customer id → update id in orders
```

```ad-warning
Use `ON DELETE CASCADE` carefully. Deleting a parent row silently deletes all child rows, which can cause unexpected data loss. `RESTRICT` (the default) is safer — it forces you to handle children explicitly.
```

---

### Performance Considerations

- Foreign keys add overhead on `INSERT`, `UPDATE`, and `DELETE` because the database must check referential integrity.
- The referenced column (parent) should be indexed — primary keys and unique columns are already indexed.
- The referencing column (child) should also be indexed for efficient JOIN and CASCADE operations.

```ad-tip
Some teams disable foreign keys in high-write-throughput systems (like analytics pipelines) and enforce integrity at the application level. This trades safety for speed — only do it when you understand the risk.
```
