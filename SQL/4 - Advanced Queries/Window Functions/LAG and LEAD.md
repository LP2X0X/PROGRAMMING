---
tags: [sql, window-functions, advanced]
---

- `LAG` and `LEAD` access values from **previous** or **next** rows without a [[Self Join]]. They're essential for comparing consecutive rows.

---

### Syntax

```sql
LAG(column, offset, default)  OVER (PARTITION BY ... ORDER BY ...)
LEAD(column, offset, default) OVER (PARTITION BY ... ORDER BY ...)
```

- `column`: the value to retrieve.
- `offset`: how many rows back (LAG) or forward (LEAD). Default is **1**.
- `default`: value to return when there's no previous/next row. Default is **NULL**.

---

### Basic Example

```sql
SELECT 
    order_date,
    total,
    LAG(total)  OVER (ORDER BY order_date) AS prev_total,
    LEAD(total) OVER (ORDER BY order_date) AS next_total
FROM orders;
```

| order_date | total | prev_total | next_total |
| ---------- | ----- | ---------- | ---------- |
| 2024-01-01 | 100   | NULL       | 250        |
| 2024-01-02 | 250   | 100        | 180        |
| 2024-01-03 | 180   | 250        | NULL       |

---

### Calculating Change

```sql
-- Day-over-day sales change
SELECT 
    order_date,
    total,
    total - LAG(total) OVER (ORDER BY order_date) AS daily_change,
    ROUND(
        100.0 * (total - LAG(total) OVER (ORDER BY order_date)) 
        / LAG(total) OVER (ORDER BY order_date), 
        1
    ) AS pct_change
FROM daily_sales;
```

---

### With PARTITION BY

```sql
-- Month-over-month change per product
SELECT 
    product_id,
    month,
    revenue,
    revenue - LAG(revenue) OVER (PARTITION BY product_id ORDER BY month) AS mom_change
FROM monthly_revenue;
```

---

### Custom Offset and Default

```sql
-- Compare to 7 days ago (offset = 7), default to 0
SELECT 
    date,
    visitors,
    LAG(visitors, 7, 0) OVER (ORDER BY date) AS visitors_last_week
FROM daily_traffic;
```

---

### FIRST_VALUE and LAST_VALUE

- Related functions that return the first or last value in the window:
```sql
SELECT 
    name, department, salary,
    FIRST_VALUE(name) OVER (
        PARTITION BY department ORDER BY salary DESC
    ) AS highest_paid_in_dept
FROM employees;
```

```ad-warning
`LAST_VALUE` with the default frame (`RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`) gives the **current row's value**, not the actual last row. Specify the full frame explicitly:
`LAST_VALUE(col) OVER (ORDER BY col ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)`
```
