---
tags: [sql, joins, querying]
---

- A **JOIN** combines rows from two or more tables based on a related column. It is arguably the most important concept in SQL after `SELECT`.
- In a properly normalized database (see [[Design]]), data is split across multiple tables to avoid redundancy. A `Users` table holds user info. An `Orders` table holds order info. To answer "what did each user order?", you **join** them together.
- Understanding JOINs is the dividing line between someone who can write basic queries and someone who can work with real databases.

---

## Why JOINs Exist

Consider two tables:

**Users**

| Id | Name | Email |
| --- | --- | --- |
| 1 | Alice | alice@mail.com |
| 2 | Bob | bob@mail.com |
| 3 | Carol | carol@mail.com |

**Orders**

| OrderId | UserId | Product | Total |
| --- | --- | --- | --- |
| 101 | 1 | Laptop | 1200 |
| 102 | 1 | Mouse | 25 |
| 103 | 2 | Keyboard | 75 |

- The `UserId` column in `Orders` is a **foreign key** pointing to `Users.Id`. This is the relationship that `JOIN` exploits.
- Without JOINs, you'd have to query each table separately and combine the data in your application code. JOINs let the database do this efficiently in a single query.

---

## Table Aliases

- Before diving into JOIN types, note that aliases are essential when joining. Without them, queries become verbose and ambiguous:

```sql
-- Without aliases (verbose)
SELECT Users.Name, Orders.Total
FROM Users
INNER JOIN Orders ON Users.Id = Orders.UserId;

-- With aliases (clean)
SELECT u.Name, o.Total
FROM Users u
INNER JOIN Orders o ON u.Id = o.UserId;
```

- The alias goes right after the table name. No `AS` keyword is needed (though `AS` is allowed).
- Use short, meaningful aliases: `u` for Users, `o` for Orders, `e` for Employees, `d` for Departments.

---

## INNER JOIN

- Returns **only** the rows where there is a match in **both** tables. Unmatched rows from either side are excluded.

```sql
SELECT u.Name, o.Product, o.Total
FROM Users u
INNER JOIN Orders o ON u.Id = o.UserId;
```

**Result:**

| Name | Product | Total |
| --- | --- | --- |
| Alice | Laptop | 1200 |
| Alice | Mouse | 25 |
| Bob | Keyboard | 75 |

- Carol is **excluded** because she has no orders (no matching row in Orders).
- `JOIN` without a prefix is shorthand for `INNER JOIN`. They are identical.

### Visual Representation

```
    Users         Orders
  ┌────────┐   ┌────────┐
  │        │   │        │
  │   ┌────┼───┼────┐   │
  │   │████│   │████│   │
  │   │████│   │████│   │
  │   └────┼───┼────┘   │
  │        │   │        │
  └────────┘   └────────┘
       INNER JOIN
   Only the overlap
```

---

## LEFT JOIN (LEFT OUTER JOIN)

- Returns **all rows from the left table**, and the matching rows from the right table. If there's no match, the right side columns are `NULL`.

```sql
SELECT u.Name, o.Product, o.Total
FROM Users u
LEFT JOIN Orders o ON u.Id = o.UserId;
```

**Result:**

| Name | Product | Total |
| --- | --- | --- |
| Alice | Laptop | 1200 |
| Alice | Mouse | 25 |
| Bob | Keyboard | 75 |
| Carol | NULL | NULL |

- Carol appears because she's in the **left** table (Users), but her Product and Total are `NULL` because she has no orders.

### Visual Representation

```
    Users         Orders
  ┌────────┐   ┌────────┐
  │████████│   │        │
  │████┌───┼───┼────┐   │
  │████│███│   │████│   │
  │████│███│   │████│   │
  │████└───┼───┼────┘   │
  │████████│   │        │
  └────────┘   └────────┘
       LEFT JOIN
  All left + matching right
```

### Use Case

- **"Show all users, even those with no orders."** This is the classic LEFT JOIN scenario.
- **Find rows with no match** — combine LEFT JOIN with `WHERE right_table.column IS NULL`:

```sql
-- Users who have NEVER placed an order
SELECT u.Name
FROM Users u
LEFT JOIN Orders o ON u.Id = o.UserId
WHERE o.OrderId IS NULL;
```

```ad-important
This `LEFT JOIN + IS NULL` pattern is one of the most useful in SQL. It answers "which rows in table A have no corresponding rows in table B?" — orphaned records, inactive users, missing data, etc.
```

---

## RIGHT JOIN (RIGHT OUTER JOIN)

- Returns **all rows from the right table**, and matching rows from the left table. If there's no match, the left side columns are `NULL`.

```sql
SELECT u.Name, o.Product, o.Total
FROM Users u
RIGHT JOIN Orders o ON u.Id = o.UserId;
```

### Visual Representation

```
    Users         Orders
  ┌────────┐   ┌────────┐
  │        │   │████████│
  │   ┌────┼───┼████████│
  │   │████│   │████████│
  │   │████│   │████████│
  │   └────┼───┼████████│
  │        │   │████████│
  └────────┘   └────────┘
       RIGHT JOIN
  Matching left + all right
```

```ad-note
`RIGHT JOIN` is rarely used in practice. Any `RIGHT JOIN` can be rewritten as a `LEFT JOIN` by swapping the table order. Most developers standardize on `LEFT JOIN` for consistency:

```sql
-- These produce the same result:
SELECT * FROM A RIGHT JOIN B ON A.id = B.a_id;
SELECT * FROM B LEFT JOIN A ON A.id = B.a_id;
```

Prefer `LEFT JOIN` — it's easier to read because the "main" table is always on the left.
```

---

## FULL OUTER JOIN

- Returns **all rows from both tables**. Where there's a match, columns from both sides are populated. Where there's no match, the missing side is `NULL`.

```sql
SELECT u.Name, o.Product, o.Total
FROM Users u
FULL OUTER JOIN Orders o ON u.Id = o.UserId;
```

- Imagine adding an order with `UserId = 99` (no matching user). FULL OUTER JOIN would show:

| Name | Product | Total |
| --- | --- | --- |
| Alice | Laptop | 1200 |
| Alice | Mouse | 25 |
| Bob | Keyboard | 75 |
| Carol | NULL | NULL |
| NULL | Mystery Item | 50 |

### Visual Representation

```
    Users         Orders
  ┌────────┐   ┌────────┐
  │████████│   │████████│
  │████┌───┼───┼████████│
  │████│███│   │████████│
  │████│███│   │████████│
  │████└───┼───┼████████│
  │████████│   │████████│
  └────────┘   └────────┘
     FULL OUTER JOIN
  All from both sides
```

```ad-warning
MySQL/MariaDB does **not** support `FULL OUTER JOIN` directly. You must simulate it with a `UNION` of `LEFT JOIN` and `RIGHT JOIN`:

```sql
SELECT u.Name, o.Product
FROM Users u LEFT JOIN Orders o ON u.Id = o.UserId
UNION
SELECT u.Name, o.Product
FROM Users u RIGHT JOIN Orders o ON u.Id = o.UserId;
```
```

---

## CROSS JOIN

- Produces the **Cartesian product** — every row from the left table paired with every row from the right table. If table A has 3 rows and table B has 4 rows, the result has 3 x 4 = 12 rows.

```sql
SELECT u.Name, p.ProductName
FROM Users u
CROSS JOIN Products p;
```

- No `ON` clause — there's no matching condition.

### When Is CROSS JOIN Useful?

- Generating all possible combinations (e.g., all sizes x all colors for a product matrix).
- Creating a calendar table by crossing years, months, and days.
- Usually **not** what you want. If you get an unexpectedly huge result set, check if you accidentally wrote a CROSS JOIN.

```ad-warning
An accidental `CROSS JOIN` (or a missing `ON` clause) is one of the most common JOIN mistakes. If your query returns millions of rows when you expected thousands, check your `ON` conditions. A `JOIN` without an `ON` clause behaves as a `CROSS JOIN`.
```

---

## All JOIN Types — Side-by-Side Summary

```
  INNER JOIN        LEFT JOIN         RIGHT JOIN        FULL OUTER JOIN
  ┌───┬───┐        ┌───┬───┐        ┌───┬───┐         ┌───┬───┐
  │   │███│        │███│███│        │   │███│         │███│███│
  │   │███│        │███│███│        │███│███│         │███│███│
  │   │███│        │███│███│        │███│███│         │███│███│
  └───┴───┘        └───┴───┘        └───┴───┘         └───┴───┘
  Only matching    All left          All right         All from both
  from both        + matching        + matching
```

| JOIN Type | Left Unmatched | Right Unmatched | Matched |
| --- | --- | --- | --- |
| INNER | Excluded | Excluded | Included |
| LEFT | Included (right = NULL) | Excluded | Included |
| RIGHT | Excluded | Included (left = NULL) | Included |
| FULL OUTER | Included (right = NULL) | Included (left = NULL) | Included |
| CROSS | N/A | N/A | Every combination |

---

## Multiple JOINs in One Query

- Real queries often join 3, 4, or more tables. Each `JOIN` adds one more table to the result:

```sql
-- Get order details with customer name and product name
SELECT 
    c.Name AS CustomerName,
    o.OrderId,
    o.OrderDate,
    p.ProductName,
    oi.Quantity,
    oi.UnitPrice,
    oi.Quantity * oi.UnitPrice AS LineTotal
FROM Customers c
INNER JOIN Orders o ON c.Id = o.CustomerId
INNER JOIN OrderItems oi ON o.OrderId = oi.OrderId
INNER JOIN Products p ON oi.ProductId = p.ProductId
ORDER BY o.OrderDate DESC;
```

- Read it as a chain: start with Customers, link to Orders, link to OrderItems, link to Products.
- You can mix JOIN types: `INNER JOIN` one table, `LEFT JOIN` another.

```sql
-- All customers with their orders (if any) and the product details
SELECT c.Name, o.OrderId, p.ProductName
FROM Customers c
LEFT JOIN Orders o ON c.Id = o.CustomerId
LEFT JOIN OrderItems oi ON o.OrderId = oi.OrderId
LEFT JOIN Products p ON oi.ProductId = p.ProductId;
```

```ad-note
When mixing `INNER JOIN` and `LEFT JOIN`, be careful with order. An `INNER JOIN` after a `LEFT JOIN` on the same chain can effectively turn the `LEFT JOIN` into an `INNER JOIN` by filtering out the NULLs. Keep your LEFT JOINs consistent down the chain, or use them only at the end.
```

---

## Self-Join

- A table joined to **itself**. This is useful when a table has a hierarchical or self-referencing relationship.

### Classic Example: Employee / Manager

| EmployeeId | Name | ManagerId |
| --- | --- | --- |
| 1 | Alice | NULL |
| 2 | Bob | 1 |
| 3 | Carol | 1 |
| 4 | Dave | 2 |

```sql
-- Show each employee and their manager's name
SELECT 
    e.Name AS Employee,
    m.Name AS Manager
FROM Employees e
LEFT JOIN Employees m ON e.ManagerId = m.EmployeeId;
```

**Result:**

| Employee | Manager |
| --- | --- |
| Alice | NULL |
| Bob | Alice |
| Carol | Alice |
| Dave | Bob |

- Aliases are **required** in self-joins to distinguish the two "copies" of the same table (`e` for employee, `m` for manager).
- Use `LEFT JOIN` so that employees without a manager (like Alice, the CEO) still appear.

---

## JOIN on Multiple Conditions

- Sometimes the relationship between tables requires more than one column:

```sql
-- Match on both product_id AND warehouse_id
SELECT oi.OrderId, p.ProductName, oi.Quantity
FROM OrderItems oi
INNER JOIN Products p 
    ON oi.ProductId = p.ProductId 
    AND oi.WarehouseId = p.WarehouseId;
```

- You can also add non-equality conditions in the `ON` clause:

```sql
-- Join orders to promotions active on the order date
SELECT o.OrderId, p.PromoName
FROM Orders o
INNER JOIN Promotions p 
    ON o.OrderDate BETWEEN p.StartDate AND p.EndDate;
```

---

## ON vs WHERE for Filter Conditions

- For `INNER JOIN`, putting conditions in `ON` vs `WHERE` produces the same result.
- For `LEFT JOIN`, it makes a **critical difference**:

```sql
-- Filter in ON: keeps all users, only joins orders > 100
SELECT u.Name, o.Total
FROM Users u
LEFT JOIN Orders o ON u.Id = o.UserId AND o.Total > 100;
-- Carol still appears (with NULL for Total)

-- Filter in WHERE: filters AFTER the join, removing NULLs
SELECT u.Name, o.Total
FROM Users u
LEFT JOIN Orders o ON u.Id = o.UserId
WHERE o.Total > 100;
-- Carol is gone (her NULL Total fails the WHERE check)
-- This effectively becomes an INNER JOIN
```

```ad-important
**Rule of thumb**: put **join conditions** in `ON`. Put **filter conditions** in `WHERE` for INNER JOIN, but in `ON` for LEFT/RIGHT JOIN if you want to preserve unmatched rows.
```

---

## Common Mistakes

### 1. Missing ON Clause

```sql
-- Missing ON → becomes a CROSS JOIN → millions of rows
SELECT u.Name, o.Total
FROM Users u
JOIN Orders o;
-- FIX: add ON u.Id = o.UserId
```

### 2. Wrong Join Column

```sql
-- Wrong: joining on unrelated columns produces garbage
SELECT u.Name, o.Total
FROM Users u
JOIN Orders o ON u.Id = o.OrderId;  -- should be o.UserId!
```

### 3. Ambiguous Column Names

```sql
-- Error: "Id" exists in both tables — which one?
SELECT Id, Name, Total
FROM Users u
JOIN Orders o ON u.Id = o.UserId;

-- Fix: qualify with table alias
SELECT u.Id, u.Name, o.Total
FROM Users u
JOIN Orders o ON u.Id = o.UserId;
```

```ad-note
Always qualify column names with table aliases when joining. Even if a column name is unique today, someone might add a same-named column to another table tomorrow, breaking your query.
```

### 4. Cartesian Product from Multiple Tables

```sql
-- WRONG: forgot to join Orders to OrderItems — gets a cross product
SELECT c.Name, o.OrderId, oi.ProductId
FROM Customers c
JOIN Orders o ON c.Id = o.CustomerId
JOIN OrderItems oi;  -- missing ON!
```

---

## Performance Notes

- Joins are fast when the join columns are **indexed**. Foreign key columns and primary keys should always have indexes.
- The database optimizer chooses the best join algorithm (nested loop, hash join, merge join) based on table sizes and available indexes.
- Adding `WHERE` conditions that reduce the number of rows **before** the join makes the join itself faster.
- Avoid joining on expressions (`ON CAST(a.Id AS VARCHAR) = b.Id`) — this prevents index usage.

```ad-note
Now that you can combine tables, learn how to aggregate and group the results: [[05 - GROUP BY and Aggregation]].
```
