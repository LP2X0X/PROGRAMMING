---
tags: [sql, set-operations, advanced]
---

- `UNION` combines the results of two or more `SELECT` statements into a single [[Result Set|result set]].

---

### UNION (Removes Duplicates)

```sql
SELECT city FROM customers
UNION
SELECT city FROM suppliers;
```

- Returns all unique cities from both tables. Duplicate rows are removed.

---

### UNION ALL (Keeps Duplicates)

```sql
SELECT city FROM customers
UNION ALL
SELECT city FROM suppliers;
```

- Returns all cities, including duplicates. **Faster** than `UNION` because it skips the deduplication step.

```ad-tip
Prefer `UNION ALL` unless you specifically need deduplication. `UNION` has to sort and compare all rows to remove duplicates, which is expensive on large [[Result Set|result sets]].
```

---

### Rules

- Both `SELECT` statements must have the **same number of columns**.
- Corresponding columns must have **compatible data types**.
- Column names in the result come from the **first** `SELECT`.

```sql
SELECT first_name AS name, 'customer' AS type FROM customers
UNION ALL
SELECT company_name, 'supplier' FROM suppliers;
-- Result columns: name, type
```

---

### ORDER BY with UNION

- `ORDER BY` applies to the **final combined result** — place it at the end:
```sql
SELECT name, city FROM customers
UNION ALL
SELECT name, city FROM suppliers
ORDER BY city, name;
```

- You cannot put `ORDER BY` inside individual `SELECT` statements within a `UNION` (unless wrapped in a subquery).

---

### Practical Example

```sql
-- Combine active and archived orders into one timeline:
SELECT order_id, customer_id, order_date, 'active' AS source
FROM orders
UNION ALL
SELECT order_id, customer_id, order_date, 'archived'
FROM archived_orders
ORDER BY order_date DESC;
```
