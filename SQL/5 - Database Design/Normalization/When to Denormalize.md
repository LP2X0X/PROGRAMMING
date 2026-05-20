---
tags: [sql, normalization, database-design]
---

- **Denormalization** intentionally adds redundancy back into a normalized database to improve read performance. It's the opposite of [[Normal Forms|normalization]].

---

### Why Denormalize?

- Normalized databases can require many JOINs to reconstruct data for a single query.
- JOINs on large tables can be slow, especially for read-heavy workloads.
- Denormalization pre-computes or duplicates data so queries are simpler and faster.

---

### Common Techniques

#### 1. Cached/Computed Columns
```sql
-- Instead of calculating order total every time:
-- SUM(item_price * quantity) FROM order_items WHERE order_id = ...

-- Store it directly:
ALTER TABLE orders ADD COLUMN total DECIMAL(10, 2);

-- Keep it updated via triggers or application logic
```

#### 2. Duplicating Columns
```sql
-- Instead of JOIN to get customer name on every order query:
ALTER TABLE orders ADD COLUMN customer_name VARCHAR(100);

-- Trade-off: must update orders.customer_name when customers.name changes
```

#### 3. Summary / Aggregate Tables
```sql
-- Pre-compute monthly sales instead of aggregating millions of order rows:
CREATE TABLE monthly_sales (
    month DATE,
    product_id INT,
    total_revenue DECIMAL(12, 2),
    total_quantity INT
);
-- Refresh periodically or via triggers
```

#### 4. Flattened Tables
- Combine multiple normalized tables into a single wide table for reporting.
- Common in **data warehouses** and **analytics** (star schema / snowflake schema).

---

### Trade-offs

| Benefit                        | Cost                                      |
| ------------------------------ | ----------------------------------------- |
| Faster reads (fewer JOINs)     | Slower writes (must update duplicates)    |
| Simpler queries                | Risk of data inconsistency                |
| Better for reporting/analytics | More storage space                        |
| Reduced query complexity       | More complex application/trigger logic    |

---

### When to Denormalize

- **After** you've normalized properly and **measured** a performance problem.
- High-traffic read queries where JOINs are the bottleneck (verify with [[EXPLAIN and Query Plans]]).
- Reporting and analytics workloads (OLAP) where data is mostly read, rarely written.
- When caching at the application level isn't sufficient.

### When NOT to Denormalize

- As a shortcut to avoid learning JOINs.
- For write-heavy workloads (denormalization makes writes slower and more complex).
- When the performance problem hasn't been measured — premature denormalization adds complexity without proven benefit.

```ad-warning
**Rule of thumb**: normalize first, denormalize only when you have measured that JOINs are the actual bottleneck. Most applications never need to denormalize — proper [[When to Use Indexes|indexing]] solves most performance problems.
```
