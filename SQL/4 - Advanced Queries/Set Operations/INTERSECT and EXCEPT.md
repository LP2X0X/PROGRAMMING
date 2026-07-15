---
tags: [sql, set-operations, advanced]
---

- `INTERSECT` and `EXCEPT` are set operations that compare the results of two queries, similar to set intersection and set difference in math.

---

### INTERSECT

- Returns only rows that appear in **both** [[Result Set|result sets]]:
```sql
SELECT customer_id FROM orders_2023
INTERSECT
SELECT customer_id FROM orders_2024;
-- Customers who placed orders in BOTH years
```

- Duplicates are removed (like `UNION`).

---

### EXCEPT (MINUS in Oracle)

- Returns rows from the **first** query that are **not** in the second query:
```sql
SELECT customer_id FROM orders_2023
EXCEPT
SELECT customer_id FROM orders_2024;
-- Customers who ordered in 2023 but NOT in 2024
```

- Order matters: `A EXCEPT B` is different from `B EXCEPT A`.

---

### Rules

- Same as [[UNION and UNION ALL]]: both queries must have the same number of columns with compatible types.
- Duplicates are automatically removed in both operations.

---

### MySQL Compatibility

```ad-note
MySQL added `INTERSECT` and `EXCEPT` support in version **8.0.31** (2022). Older versions don't support them. Use alternatives:
```

```sql
-- INTERSECT alternative using INNER JOIN:
SELECT DISTINCT a.customer_id
FROM orders_2023 a
INNER JOIN orders_2024 b ON a.customer_id = b.customer_id;

-- INTERSECT alternative using EXISTS:
SELECT DISTINCT customer_id FROM orders_2023
WHERE customer_id IN (SELECT customer_id FROM orders_2024);

-- EXCEPT alternative using LEFT JOIN:
SELECT DISTINCT a.customer_id
FROM orders_2023 a
LEFT JOIN orders_2024 b ON a.customer_id = b.customer_id
WHERE b.customer_id IS NULL;

-- EXCEPT alternative using NOT EXISTS:
SELECT DISTINCT customer_id FROM orders_2023
WHERE customer_id NOT IN (SELECT customer_id FROM orders_2024);
```

```ad-tip
When both `INTERSECT`/`EXCEPT` and JOIN-based alternatives are available, the optimizer usually produces the same plan. Choose whichever reads more clearly. For checking "exists in both" or "exists only in one," the set operations are often more readable.
```
