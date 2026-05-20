---
tags: [sql, fundamentals, term]
---

- Choosing the right data type matters for storage efficiency, query performance, and data integrity.

---

### Numeric Types

| Type            | Description                                    |
| --------------- | ---------------------------------------------- |
| `INT`           | Whole numbers. 4 bytes, range ±2.1 billion.    |
| `BIGINT`        | Large whole numbers. 8 bytes.                  |
| `SMALLINT`      | Smaller range. 2 bytes.                        |
| `TINYINT`       | 0–255 (MySQL) or -128–127. 1 byte.            |
| `DECIMAL(p, s)` | Exact numeric. `p` = total digits, `s` = decimal places. Use for money. |
| `FLOAT` / `DOUBLE` | Approximate floating-point. Faster but imprecise — never use for money. |

```ad-warning
`FLOAT` and `DOUBLE` can produce rounding errors. Always use `DECIMAL` for financial or precision-critical data.
```

---

### String Types

| Type           | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| `VARCHAR(n)`   | Variable-length string up to `n` characters. Most commonly used string type. |
| `CHAR(n)`      | Fixed-length string, padded with spaces. Use for fixed-width data (e.g., country codes). |
| `TEXT`          | Variable-length with very large max. Cannot be indexed fully in some RDBMS. |

- Prefer `VARCHAR` over `CHAR` unless the length is truly fixed.
- Prefer `VARCHAR` over `TEXT` when you can define a reasonable max length — it allows indexing and constraint checks.

---

### Date / Time Types

| Type          | Description                                   |
| ------------- | --------------------------------------------- |
| `DATE`        | Date only (YYYY-MM-DD).                       |
| `TIME`        | Time only (HH:MM:SS).                         |
| `DATETIME`    | Date and time, no timezone awareness.          |
| `TIMESTAMP`   | Date and time, often stored as UTC. Auto-updates in some RDBMS. |

```ad-tip
Always store dates in `DATE`/`DATETIME`/`TIMESTAMP` types — never as strings. String dates break sorting, comparison, and date arithmetic. See [[Date Functions]].
```

---

### Other Types

| Type      | Description                                           |
| --------- | ----------------------------------------------------- |
| `BOOLEAN` | `TRUE` / `FALSE`. In MySQL, stored as `TINYINT(1)`.  |
| `BLOB`    | Binary Large Object — for images, files, binary data. |
| `JSON`    | Native JSON storage (MySQL 5.7+, PostgreSQL).         |
| `ENUM`    | A column restricted to a predefined set of values (MySQL). |
