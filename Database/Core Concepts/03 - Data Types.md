---
tags: [database, fundamentals, data-types]
---

**Prerequisite:** [[02 - Tables, Rows, and Columns]]

- Every column in a table has a **data type** — it defines what kind of values the column can hold. The DBMS enforces the type on every insert and update.
- You define the data type when creating the table. Once set, every row must conform to it.
- Data types vary by database system, but the core categories are universal: **numeric**, **string**, **date/time**, **binary**, and **boolean**.

---

### Numeric Types

| Type | SQL Server | MySQL / MariaDB | Size | Range / Description |
| --- | --- | --- | --- | --- |
| Tiny integer | `TINYINT` | `TINYINT` | 1 byte | 0 to 255 (SQL Server unsigned); -128 to 127 (MySQL signed) |
| Small integer | `SMALLINT` | `SMALLINT` | 2 bytes | -32,768 to 32,767 |
| Integer | `INT` | `INT` | 4 bytes | -2,147,483,648 to 2,147,483,647 (~2.1 billion) |
| Big integer | `BIGINT` | `BIGINT` | 8 bytes | -9.2 quintillion to 9.2 quintillion |
| Decimal (exact) | `DECIMAL(p, s)` | `DECIMAL(p, s)` | Varies | Fixed precision. `p` = total digits, `s` = digits after decimal point. Use for money, measurements, anything requiring exact math. |
| Numeric (exact) | `NUMERIC(p, s)` | `NUMERIC(p, s)` | Varies | Functionally identical to `DECIMAL` in most systems |
| Float (approx) | `FLOAT` | `FLOAT` | 4 bytes | Approximate floating-point. ~7 significant digits. |
| Double (approx) | `FLOAT(53)` | `DOUBLE` | 8 bytes | Approximate floating-point. ~15 significant digits. |
| Bit / Boolean | `BIT` | `TINYINT(1)` / `BOOL` | 1 bit / 1 byte | 0 or 1 (SQL Server); 0 or 1 but stored as a byte (MySQL). `BOOL` in MySQL is an alias for `TINYINT(1)`. |
| Money | `MONEY` | N/A | 8 bytes | SQL Server specific. Stores currency values. Not recommended — use `DECIMAL(19, 4)` instead for portability. |

```ad-warning
title: Never Use FLOAT for Money
`FLOAT` and `DOUBLE` use IEEE 754 floating-point representation, which **cannot exactly represent** many decimal fractions. Classic example:

- `0.1 + 0.2 = 0.30000000000000004` (not `0.3`)

This leads to rounding errors that accumulate over time. For financial data, **always use `DECIMAL`**:

- `DECIMAL(10, 2)` — up to 10 digits total, 2 after the decimal point. Stores values like `99999999.99` exactly.
- `DECIMAL(19, 4)` — common choice for currency when you need sub-cent precision (e.g., exchange rates).
```

#### Choosing the Right Numeric Type

- Use the **smallest type that fits your data**:
  - An `Age` column? `TINYINT` (0-255) is more than enough — not `INT`.
  - A `Quantity` column for a store? `SMALLINT` or `INT`.
  - A table that will never have 2 billion rows? `INT` for the primary key — not `BIGINT`.
- `BIGINT` is 8 bytes per row. On a table with 100 million rows and 3 `BIGINT` columns, that's 2.4 GB just for those columns. Use it only when you genuinely need the range.

```ad-note
title: DECIMAL Precision and Scale
`DECIMAL(p, s)` — `p` is **precision** (total number of digits), `s` is **scale** (digits after the decimal point).

- `DECIMAL(5, 2)` can store from `-999.99` to `999.99`
- `DECIMAL(10, 0)` is effectively a 10-digit integer
- `DECIMAL(18, 6)` can store values like `123456789012.123456`

If you insert a value with more decimal places than the scale allows, the DBMS **rounds** it (in most systems). If you insert a value with more integer digits than `p - s` allows, the DBMS **rejects** it with an overflow error.
```

---

### String Types

| Type | SQL Server | MySQL / MariaDB | Description |
| --- | --- | --- | --- |
| Fixed-length | `CHAR(n)` | `CHAR(n)` | Always stores exactly `n` characters. Shorter values are **padded with spaces**. Max `n`: 8,000 (SQL Server), 255 (MySQL). |
| Variable-length | `VARCHAR(n)` | `VARCHAR(n)` | Stores up to `n` characters. No padding — only uses the space needed. Max `n`: 8,000 (SQL Server), 65,535 (MySQL). |
| Unicode fixed | `NCHAR(n)` | N/A | Like `CHAR` but stores Unicode (UTF-16). 2 bytes per character. SQL Server specific. |
| Unicode variable | `NVARCHAR(n)` | N/A | Like `VARCHAR` but stores Unicode (UTF-16). SQL Server specific. MySQL's `VARCHAR` is already UTF-8 by default. |
| Large text | `VARCHAR(MAX)` | `TEXT` / `MEDIUMTEXT` / `LONGTEXT` | Very large strings. `VARCHAR(MAX)` stores up to 2 GB (SQL Server). `LONGTEXT` stores up to 4 GB (MySQL). |

#### CHAR vs VARCHAR

| | `CHAR(n)` | `VARCHAR(n)` |
| --- | --- | --- |
| **Storage** | Always `n` bytes (+ overhead) | Actual length + 1-2 bytes for length prefix |
| **Padding** | Right-padded with spaces to fill `n` | No padding |
| **When to use** | Fixed-width data: country codes (`CHAR(2)`), state codes (`CHAR(2)`), MD5 hashes (`CHAR(32)`) | Everything else — names, emails, descriptions |
| **Comparison** | Trailing spaces may be ignored in comparisons (depends on collation) | Exact comparison |

```ad-tip
Default to `VARCHAR` unless you have a genuine fixed-width value. `CHAR(100)` for a name field wastes space on every row that is shorter than 100 characters.
```

#### Unicode in SQL Server

- SQL Server has separate types for ASCII (`CHAR`, `VARCHAR`) and Unicode (`NCHAR`, `NVARCHAR`):
  - `VARCHAR(100)` — 1 byte per character, ASCII/extended ASCII only
  - `NVARCHAR(100)` — 2 bytes per character, supports all Unicode characters (Chinese, Arabic, emoji, etc.)
- MySQL and MariaDB default to UTF-8 encoding for `VARCHAR`, so Unicode is supported without a separate type.

```ad-important
In SQL Server, if your application handles any non-English text (user names, addresses, product descriptions), **always use `NVARCHAR`**. Using `VARCHAR` for Unicode data silently corrupts characters — a `VARCHAR` column will store `?` instead of `ö`, `ñ`, or `漢`.
```

- When inserting Unicode string literals in SQL Server, prefix with `N`:

```sql
-- Correct: N prefix for Unicode
INSERT INTO Users (Name) VALUES (N'Müller');

-- Wrong: without N prefix, ü may be corrupted
INSERT INTO Users (Name) VALUES ('Müller');
```

---

### Date and Time Types

| Type | SQL Server | MySQL / MariaDB | Description |
| --- | --- | --- | --- |
| Date only | `DATE` | `DATE` | `YYYY-MM-DD`. No time component. Range: 0001-01-01 to 9999-12-31 (SQL Server). |
| Time only | `TIME` | `TIME` | `HH:MM:SS[.fractional]`. No date component. |
| Date + time | `DATETIME2(n)` | `DATETIME` | Full timestamp. `n` is fractional seconds precision (0-7 in SQL Server). |
| Legacy date+time | `DATETIME` | N/A | SQL Server legacy type. Lower precision (3.33ms) and smaller range than `DATETIME2`. Avoid in new code. |
| With timezone | `DATETIMEOFFSET` | `TIMESTAMP` | Timezone-aware. `DATETIMEOFFSET` stores UTC offset. MySQL's `TIMESTAMP` auto-converts to/from UTC based on server timezone. |
| Small date+time | `SMALLDATETIME` | N/A | SQL Server. Minute precision only, range 1900-2079. Saves space but limited. |

```ad-warning
title: DATETIME vs DATETIME2 in SQL Server
Always use `DATETIME2` in new SQL Server databases, not `DATETIME`:

- `DATETIME` has only 3.33ms precision and range 1753-9999
- `DATETIME2` has up to 100ns precision and range 0001-9999
- `DATETIME2` uses less storage for equivalent precision
- `DATETIME` is a legacy type kept for backward compatibility
```

```ad-tip
title: Store Dates as Date Types, Never as Strings
Never store dates as `VARCHAR` (e.g., `'2026-06-03'` as a string). String dates:

- Cannot be compared correctly — `'9/1/2026' > '10/1/2026'` is `TRUE` (string comparison)
- Break date arithmetic — you cannot do `DATEADD(day, 7, '2026-06-03')` on a string
- Lose timezone handling
- Waste storage space

Always use the appropriate `DATE` / `DATETIME2` / `TIMESTAMP` type. See [[Date Functions]] for operations on date types.
```

---

### Binary Types

| Type | SQL Server | MySQL / MariaDB | Description |
| --- | --- | --- | --- |
| Fixed binary | `BINARY(n)` | `BINARY(n)` | Fixed-length binary data, padded with `0x00` |
| Variable binary | `VARBINARY(n)` | `VARBINARY(n)` | Variable-length binary data up to `n` bytes |
| Large binary | `VARBINARY(MAX)` | `BLOB` / `MEDIUMBLOB` / `LONGBLOB` | Large binary objects — up to 2 GB (SQL Server), 4 GB (MySQL) |

- Binary types store raw bytes — images, files, encrypted data, hashes.
- In practice, large files (images, documents) are usually stored in the **file system** or **object storage** (S3, Azure Blob), with only a file path or URL stored in the database. Storing large blobs in the database increases backup size, slows queries, and wastes buffer pool memory.

```ad-tip
Store small binary values in the database (e.g., password hashes at 32-64 bytes, thumbnails under 256 KB). Store large files externally and keep a reference (URL or file path) in the database.
```

---

### Other Notable Types

| Type | SQL Server | MySQL / MariaDB | Description |
| --- | --- | --- | --- |
| GUID / UUID | `UNIQUEIDENTIFIER` | `CHAR(36)` or `BINARY(16)` | Globally unique identifier. 16 bytes. See [[04 - Primary Keys and Unique Identifiers]]. |
| JSON | `NVARCHAR(MAX)` (no native JSON type) | `JSON` (MySQL 5.7+) | Stores JSON documents. MySQL validates JSON on insert; SQL Server stores it as a string but has JSON functions. |
| XML | `XML` | N/A | SQL Server has a native XML type with XQuery support. MySQL has no native XML type. |
| Spatial | `GEOMETRY` / `GEOGRAPHY` | `GEOMETRY` / `POINT` | Geographic coordinates, shapes. Used in mapping and GIS applications. |

---

### Choosing the Right Data Type — Decision Guide

```ad-important
title: Golden Rules for Data Type Selection
1. **Use the smallest type that fits** — smaller types mean more rows per page, better cache utilization, faster queries
2. **Use `DECIMAL` for money** — never `FLOAT`
3. **Use date types for dates** — never `VARCHAR`
4. **Use `VARCHAR` over `CHAR`** — unless the length is truly fixed
5. **Use `NVARCHAR` in SQL Server for international text** — `VARCHAR` corrupts non-ASCII characters
6. **Use `INT` for primary keys** — unless you genuinely need `BIGINT` (billions of rows) or `UNIQUEIDENTIFIER` (distributed systems)
7. **Don't over-size columns** — `VARCHAR(8000)` for an email address wastes metadata and misleads developers about expected values
```

#### Common Column → Type Mappings

| Column purpose | Recommended type | Why |
| --- | --- | --- |
| Primary key | `INT IDENTITY` / `INT AUTO_INCREMENT` | Compact, fast, sequential. See [[04 - Primary Keys and Unique Identifiers]]. |
| Name, email, address | `NVARCHAR(n)` / `VARCHAR(n)` | Variable-length text. `NVARCHAR` if Unicode needed. |
| Price, salary, balance | `DECIMAL(19, 4)` or `DECIMAL(10, 2)` | Exact decimal math. No rounding errors. |
| Age, quantity, count | `TINYINT` / `SMALLINT` / `INT` | Smallest integer type that fits the range. |
| True/false flag | `BIT` / `TINYINT(1)` | `IsActive`, `HasDiscount`, etc. |
| Created/updated timestamp | `DATETIME2` / `DATETIME` / `TIMESTAMP` | Automatic tracking of record creation/modification. |
| Country code | `CHAR(2)` | Fixed-width: `'US'`, `'VN'`, `'JP'` |
| Description, comments | `NVARCHAR(MAX)` / `TEXT` | Large variable-length text |
| File reference | `VARCHAR(500)` | URL or file path, not the file itself |

---

### Type Conversion and Casting

- Sometimes you need to convert between types — for example, converting a number to a string for concatenation, or parsing a string as a date.
- SQL provides `CAST` and `CONVERT` functions for this:

```sql
-- CAST (standard SQL)
SELECT CAST(Price AS VARCHAR(20)) FROM Products;
SELECT CAST('2026-06-03' AS DATE);

-- CONVERT (SQL Server specific, with style codes)
SELECT CONVERT(VARCHAR(10), GETDATE(), 120);  -- '2026-06-03'
```

- See [[CAST and CONVERT]] for detailed coverage.

```ad-warning
title: Implicit Conversion Pitfalls
The DBMS can sometimes convert types automatically (implicit conversion), but this can cause subtle bugs and performance problems:

- Comparing `NVARCHAR` to `VARCHAR` forces conversion on every row — kills index usage
- Comparing `INT` to `VARCHAR` may succeed or fail unpredictably depending on the string values
- Inserting `'123abc'` into an `INT` column fails — no implicit conversion possible

Always use explicit types and explicit `CAST` / `CONVERT` when types don't match.
```

---

**Next:** [[04 - Primary Keys and Unique Identifiers]]
