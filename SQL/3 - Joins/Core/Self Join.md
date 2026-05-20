---
tags: [sql, joins]
---

- A **self join** is when a table is joined **to itself**. You must use table aliases to distinguish the two "copies" of the same table.

---

### Classic Example: Employee → Manager

```sql
-- employees table has: id, name, manager_id (references id in the same table)

SELECT 
    e.name AS employee,
    m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

| employee | manager |
| -------- | ------- |
| Alice    | Bob     |
| Bob      | Carol   |
| Carol    | NULL    |

- Carol has no manager (she's the CEO), so `LEFT JOIN` preserves her row with NULL.
- Using [[INNER JOIN]] instead would exclude Carol from the result.

---

### Comparing Rows in the Same Table

```sql
-- Find pairs of employees in the same department
SELECT 
    a.name AS employee_1,
    b.name AS employee_2,
    a.department
FROM employees a
INNER JOIN employees b 
    ON a.department = b.department 
    AND a.id < b.id;  -- avoid duplicates and self-pairing
```

- `a.id < b.id` ensures each pair appears only once and an employee isn't paired with themselves.

---

### Finding Sequential Records

```sql
-- Find consecutive orders by the same customer
SELECT 
    o1.order_id AS first_order,
    o2.order_id AS next_order,
    o1.customer_id
FROM orders o1
INNER JOIN orders o2 
    ON o1.customer_id = o2.customer_id
    AND o2.order_date = (
        SELECT MIN(order_date) 
        FROM orders 
        WHERE customer_id = o1.customer_id 
          AND order_date > o1.order_date
    );
```

```ad-tip
For sequential row comparisons, [[LAG and LEAD]] window functions are often simpler and more performant than self joins.
```
