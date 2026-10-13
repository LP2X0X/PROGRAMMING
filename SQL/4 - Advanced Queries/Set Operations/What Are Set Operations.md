---
tags: [sql, set-operations, advanced]
---

Set operations combine the **results** of two or more `SELECT` queries into a single result set. They treat each query's output as a set of rows and apply set theory operations on them.

### The Four Set Operations

| Operation | What it returns |
|---|---|
| `UNION` | All rows from both queries (duplicates removed) |
| `UNION ALL` | All rows from both queries (duplicates kept) |
| `INTERSECT` | Only rows that appear in **both** queries |
| `EXCEPT` | Rows from the first query that are **not** in the second |

```sql
-- Query A returns:  {1, 2, 3}
-- Query B returns:  {2, 3, 4}

A UNION B      → {1, 2, 3, 4}
A UNION ALL B  → {1, 2, 3, 2, 3, 4}
A INTERSECT B  → {2, 3}
A EXCEPT B     → {1}
B EXCEPT A     → {4}
```

### How They Differ from Joins

A join combines **columns** — it matches rows from two tables horizontally based on a condition:

```sql
SELECT * FROM A INNER JOIN B ON A.id = B.id
-- Result: columns from A + columns from B, matched by id
```

A set operation combines **rows** — it stacks two result sets vertically with no matching condition:

```sql
SELECT name FROM A
UNION
SELECT name FROM B
-- Result: all names from A, then all names from B (same columns)
```

Think of it as: **joins go sideways, set operations go downward.**

```ad-tip
A set operation is essentially a join without an `ON` clause — you're combining data from multiple sources, but by stacking rows rather than linking columns. There's no relationship being established between the tables; you just need two queries that produce the same shape of output.
```

### Requirements

- Both `SELECT` statements must have the **same number of columns**
- Corresponding columns must have **compatible data types**
- Column names in the result come from the **first** `SELECT`

See [[UNION and UNION ALL]] and [[INTERSECT and EXCEPT]] for details on each operation.
