---
tags: [sql, fundamentals, term]
---

- SQL stands for **Structured Query Language**. It is the standard language for interacting with relational databases.
- SQL is **case-insensitive** for keywords, but the convention is to write keywords in UPPERCASE and identifiers (table/column names) in lowercase:
```sql
SELECT first_name, last_name FROM employees WHERE age > 30;
```
- Every SQL statement ends with a **semicolon** (`;`).

---

### Comments

```sql
-- This is a single-line comment

/* This is a
   multi-line comment */
```

---

### SQL Statement Categories

| Category | Name                          | Purpose                              | Key Statements                          |
| -------- | ----------------------------- | ------------------------------------ | --------------------------------------- |
| **DDL**  | Data Definition Language      | Define/modify database structure     | `CREATE`, `ALTER`, `DROP`, `TRUNCATE`   |
| **DML**  | Data Manipulation Language    | Read and modify data                 | `SELECT`, `INSERT`, `UPDATE`, `DELETE`  |
| **DCL**  | Data Control Language         | Manage permissions                   | `GRANT`, `REVOKE`                       |
| **TCL**  | Transaction Control Language  | Manage transactions                  | `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT` |

- The most frequently used category is **DML** — especially [[SELECT]], [[INSERT]], [[UPDATE]], and [[DELETE]].

---

### Identifiers and Quoting

- Table and column names should avoid SQL reserved words (`SELECT`, `ORDER`, `GROUP`, etc.).
- If you must use a reserved word as an identifier, quote it:
  - MySQL: backticks — `` `order` ``
  - PostgreSQL / SQL Server: double quotes — `"order"`
  - SQL Server also supports brackets — `[order]`

---

### Execution Order

- SQL statements are **not** executed in the order they are written. The logical execution order is:
  1. `FROM` / `JOIN`
  2. `WHERE`
  3. `GROUP BY`
  4. `HAVING`
  5. `SELECT`
  6. `DISTINCT`
  7. `ORDER BY`
  8. `LIMIT` / `OFFSET`

```ad-note
This is why you cannot use a [[Column Alias|column alias]] defined in `SELECT` inside a `WHERE` clause — `WHERE` is evaluated before `SELECT`.
```
