---
tags: [sql, dml, reading-data]
---

- The `WHERE` clause filters rows before they appear in the result. Only rows that satisfy the condition are returned.

---

### Basic Syntax

```sql
SELECT * FROM employees
WHERE department = 'Engineering';
```

---

### Comparison Operators

| Operator | Meaning                |
| -------- | ---------------------- |
| `=`      | Equal to               |
| `<>` or `!=` | Not equal to     |
| `<`      | Less than              |
| `>`      | Greater than           |
| `<=`     | Less than or equal     |
| `>=`     | Greater than or equal  |

---

### Logical Operators

- **AND**: both conditions must be true.
- **OR**: at least one condition must be true.
- **NOT**: negates the condition.

```sql
SELECT * FROM employees
WHERE department = 'Engineering' AND salary > 70000;

SELECT * FROM employees
WHERE department = 'Engineering' OR department = 'Marketing';

SELECT * FROM employees
WHERE NOT department = 'Sales';
```

---

### Operator Precedence

- `NOT` is evaluated first, then `AND`, then `OR`.
- This can lead to unexpected results:
```sql
-- This might NOT do what you expect:
SELECT * FROM employees
WHERE department = 'Engineering' OR department = 'Marketing' AND salary > 80000;

-- AND binds tighter, so this actually means:
-- department = 'Engineering' OR (department = 'Marketing' AND salary > 80000)
```

```ad-tip
Always use **parentheses** to make your intent explicit, especially when mixing `AND` and `OR`:
```

```sql
SELECT * FROM employees
WHERE (department = 'Engineering' OR department = 'Marketing')
  AND salary > 80000;
```

---

### WHERE with Different Data Types

```sql
WHERE age = 30              -- numeric: no quotes
WHERE name = 'Alice'        -- string: single quotes
WHERE hire_date = '2024-01-15'  -- date: single quotes, ISO format
WHERE active = TRUE         -- boolean
```

```ad-note
`WHERE` is evaluated **before** `SELECT`, which means you cannot filter on a column alias defined in `SELECT`. Use [[HAVING]] for filtering after aggregation, or wrap in a subquery. See [[SQL Syntax Basics]] for the full execution order.
```
