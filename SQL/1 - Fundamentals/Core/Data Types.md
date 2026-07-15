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
| `TEXT`          | Variable-length up to 65,535 bytes (~64 KB). Cannot be indexed fully in some RDBMS. |
| `MEDIUMTEXT`   | Up to 16,777,215 bytes (~16 MB). Use for long articles, HTML content, or serialized data. MySQL/MariaDB only. |
| `LONGTEXT`     | Up to 4,294,967,295 bytes (~4 GB). Rarely needed — `MEDIUMTEXT` covers most cases. MySQL/MariaDB only. |

- Prefer `VARCHAR` over `CHAR` unless the length is truly fixed.
- Prefer `VARCHAR` over `TEXT` when you can define a reasonable max length — it allows indexing and constraint checks.

```ad-note
title: CHAR vs VARCHAR
Both `CHAR(n)` and `VARCHAR(n)` hold a maximum of `n` characters and reject anything longer.

- `CHAR(n)` — always stores exactly `n` bytes. Shorter values are **right-padded with spaces**. `CHAR(10)` storing `'hi'` → `'hi        '` (8 trailing spaces).
- `VARCHAR(n)` — stores only the actual characters + 1–2 bytes for a length prefix. `VARCHAR(10)` storing `'hi'` → 2 bytes + length prefix.

Use `CHAR` for truly fixed-width data (country codes `CHAR(2)`, state codes `CHAR(2)`, MD5 hashes `CHAR(32)`). Use `VARCHAR` for everything else.
```

---

### Date / Time Types

| Type          | Description                                   |
| ------------- | --------------------------------------------- |
| `DATE`        | Date only (YYYY-MM-DD).                       |
| `TIME`        | Time only (HH:MM:SS).                         |
| `DATETIME`    | Date and time. 8 bytes. Range: `1000-01-01` to `9999-12-31`. Stores the literal value as-is — no timezone conversion. |
| `TIMESTAMP`   | Date and time. 4 bytes. Range: `1970-01-01` to `2038-01-19`. Converts to UTC on storage, converts back to session timezone on retrieval. |

```ad-tip
Always store dates in `DATE`/`DATETIME`/`TIMESTAMP` types — never as strings. String dates break sorting, comparison, and date arithmetic. See [[Date Functions]].
```

#### DATETIME vs TIMESTAMP

|                    | `DATETIME`                          | `TIMESTAMP`                                |
| ------------------ | ----------------------------------- | ------------------------------------------ |
| **Storage**        | 8 bytes, stores literal value       | 4 bytes, stores seconds since Unix epoch   |
| **Range**          | `1000-01-01` to `9999-12-31`        | `1970-01-01` to `2038-01-19` (2038 problem) |
| **Timezone**       | No conversion — what you insert is what you get back | Converts to UTC on write, back to session timezone on read |
| **Use for**        | Fixed points in time: birthdates, scheduled appointments, historical dates | Event times that should be timezone-aware: `created_at`, `updated_at`, login times |

---

### Other Types

| Type      | Description                                           |
| --------- | ----------------------------------------------------- |
| `BOOLEAN` | `TRUE` / `FALSE`. In MySQL, stored as `TINYINT(1)`.  |
| `BLOB`    | Binary Large Object — for images, files, binary data. |
| `JSON`    | Native JSON storage (MySQL 5.7+, PostgreSQL).         |
| `ENUM`    | A column restricted to a predefined set of values (MySQL). Similar to a [[Unique, Not Null, Check, Default#CHECK|CHECK constraint]] but defined as a type. |
