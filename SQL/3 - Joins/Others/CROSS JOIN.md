---
tags: [sql, joins]
---

- A `CROSS JOIN` produces the **Cartesian product** — every row from the first table combined with every row from the second table.
- No `ON` clause is needed (there's no matching condition).

---

### Syntax

```sql
SELECT s.size, c.color
FROM sizes s
CROSS JOIN colors c;
```

If `sizes` has 3 rows and `colors` has 4 rows, the result has **3 × 4 = 12 rows**.

| size   | color |
| ------ | ----- |
| Small  | Red   |
| Small  | Blue  |
| Small  | Green |
| Small  | Black |
| Medium | Red   |
| Medium | Blue  |
| ...    | ...   |

---

### Implicit Cross Join

```sql
-- Older syntax (comma-separated FROM) also produces a cross join:
SELECT s.size, c.color
FROM sizes s, colors c;
```

- This is equivalent but less explicit. Prefer `CROSS JOIN` for clarity.

---

### Use Cases

- **Generating combinations**: all size/color variants, all date/product pairs.
- **Creating a calendar grid**: cross join days × time slots.
- **Pairing every row with a reference set**: every employee × every training course.

```sql
-- Generate a report template with all dates and all products
SELECT d.date, p.product_name, 0 AS sales
FROM dates d
CROSS JOIN products p;
```

```ad-warning
Cross joins can produce **very large result sets**. A cross join of two 1,000-row tables produces 1,000,000 rows. Always be aware of the table sizes before using `CROSS JOIN`.
```
