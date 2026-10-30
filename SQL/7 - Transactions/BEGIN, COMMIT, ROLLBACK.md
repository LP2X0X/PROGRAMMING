---
tags: [sql, transactions]
---

- These three commands control transaction boundaries — when a transaction starts, when it's saved, and when it's undone.

---

### Basic Transaction Flow

```sql
-- MySQL / PostgreSQL:
START TRANSACTION;

UPDATE accounts SET balance = balance - 500 WHERE id = 1;
UPDATE accounts SET balance = balance + 500 WHERE id = 2;

COMMIT;   -- make changes permanent
```

```sql
-- If something goes wrong:
START TRANSACTION;

UPDATE accounts SET balance = balance - 500 WHERE id = 1;
-- oops, target account doesn't exist
ROLLBACK;  -- undo everything
```

---

### Syntax by RDBMS

| RDBMS         | Start a transaction                   |
| ------------- | ------------------------------------- |
| MySQL         | `START TRANSACTION;` or `BEGIN;`      |
| PostgreSQL    | `BEGIN;` or `START TRANSACTION;`      |
| SQL Server    | `BEGIN TRANSACTION;`                  |
| SQLite        | `BEGIN TRANSACTION;`                  |

- `COMMIT` and `ROLLBACK` are the same across all RDBMS.

---

### Auto-Commit Mode

- By default, most databases run in **auto-commit mode** — every individual statement is automatically wrapped in its own transaction and committed immediately.
- `START TRANSACTION` temporarily disables auto-commit until you `COMMIT` or `ROLLBACK`.

```sql
-- These two are equivalent in auto-commit mode:
INSERT INTO logs (msg) VALUES ('hello');

-- Is the same as:
START TRANSACTION;
INSERT INTO logs (msg) VALUES ('hello');
COMMIT;
```

---

### SAVEPOINT

- Creates a **checkpoint** within a transaction that you can roll back to without undoing the entire transaction:
```sql
START TRANSACTION;

INSERT INTO orders (customer_id, total) VALUES (1, 100.00);
SAVEPOINT after_order;

INSERT INTO order_items (order_id, product_id) VALUES (1, 999);  -- fails: product doesn't exist
ROLLBACK TO SAVEPOINT after_order;  -- undo only the failed item insert

-- The order insert is still intact
INSERT INTO order_items (order_id, product_id) VALUES (1, 42);  -- this one works
COMMIT;
```

- `RELEASE SAVEPOINT name` — removes a savepoint (the changes are kept).

---

### Practical Pattern: Error Handling

```sql
-- MySQL stored procedure pattern:
DELIMITER //
CREATE PROCEDURE transfer_funds(
    IN from_id INT, IN to_id INT, IN amount DECIMAL(10,2)
)
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
    END;
    
    START TRANSACTION;
    UPDATE accounts SET balance = balance - amount WHERE id = from_id;
    UPDATE accounts SET balance = balance + amount WHERE id = to_id;
    COMMIT;
END //
DELIMITER ;
```

```ad-tip
Keep transactions **short**. Long-running transactions hold locks and block other users. Do your data preparation outside the transaction, then wrap only the critical writes in BEGIN/COMMIT. See [[ACID Properties]] and [[Isolation Levels]].
```
