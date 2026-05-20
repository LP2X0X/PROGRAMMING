---
tags: [sql, window-functions, advanced]
---

- Any [[Aggregate Functions|aggregate function]] (`SUM`, `AVG`, `COUNT`, `MIN`, `MAX`) can be used as a window function by adding an `OVER()` clause. The key difference: rows are **not collapsed**.

---

### Running Total

```sql
SELECT 
    order_date,
    total,
    SUM(total) OVER (ORDER BY order_date) AS running_total
FROM orders;
```

| order_date | total | running_total |
| ---------- | ----- | ------------- |
| 2024-01-01 | 100   | 100           |
| 2024-01-02 | 250   | 350           |
| 2024-01-03 | 180   | 530           |

---

### Running Total Per Group

```sql
SELECT 
    customer_id,
    order_date,
    total,
    SUM(total) OVER (
        PARTITION BY customer_id ORDER BY order_date
    ) AS customer_running_total
FROM orders;
```

---

### Moving Average

```sql
-- 3-day moving average
SELECT 
    date,
    revenue,
    AVG(revenue) OVER (
        ORDER BY date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3d
FROM daily_revenue;
```

- `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW` means: current row + the 2 rows before it.

---

### Count Per Group (Without Collapsing)

```sql
SELECT 
    name, department,
    COUNT(*) OVER (PARTITION BY department) AS dept_size
FROM employees;
-- Every employee shows how many people are in their department
```

---

### Percentage of Total

```sql
SELECT 
    department,
    salary,
    salary * 100.0 / SUM(salary) OVER () AS pct_of_total,
    salary * 100.0 / SUM(salary) OVER (PARTITION BY department) AS pct_of_dept
FROM employees;
```

- `OVER ()` (empty) = entire result set as one partition.
- `OVER (PARTITION BY department)` = within each department.

---

### Regular Aggregate vs Window Aggregate

```sql
-- Regular: 5 departments → 5 rows
SELECT department, SUM(salary) FROM employees GROUP BY department;

-- Window: all employees → all rows preserved, sum added alongside
SELECT name, department, salary,
       SUM(salary) OVER (PARTITION BY department) AS dept_total
FROM employees;
```

```ad-tip
Window aggregates are perfect for calculations like "each row's percentage of its group total" or "running cumulative sum" — things that are awkward or impossible with plain GROUP BY. See [[Window Functions Overview]] for the general syntax.
```
