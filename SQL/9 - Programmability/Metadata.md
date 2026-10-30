---
tags: [sql, programmability, metadata]
---

- **Metadata** is "data about data" — information about the database itself rather than the data stored in it. This includes table names, column names, data types, constraints, indexes, views, users, and privileges.
- Every database server stores metadata in a set of internal tables called the **data dictionary** (also called the **system catalog**). You can query this metadata just like regular data.

---

### Why Metadata Matters

| Use case | Example |
|---|---|
| **Explore an unfamiliar database** | "What tables exist? What columns does this table have?" |
| **Generate dynamic SQL** | Build queries programmatically by reading table/column names |
| **Admin and monitoring** | Check index usage, constraint definitions, privileges |
| **Schema validation** | Verify a column exists before running a migration script |
| **Documentation** | Auto-generate data dictionaries from live schema |

---

### INFORMATION_SCHEMA — The Standard Way

- `INFORMATION_SCHEMA` is a read-only virtual database defined by the SQL standard. It provides a consistent way to query metadata across different database systems.
- You query it with regular `SELECT` statements:

```sql
-- List all tables in the current database
SELECT TABLE_NAME, TABLE_TYPE
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = DATABASE()
ORDER BY TABLE_NAME;

-- List all columns in a specific table
SELECT COLUMN_NAME, DATA_TYPE, IS_NULLABLE, COLUMN_DEFAULT
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = DATABASE()
  AND TABLE_NAME = 'customer'
ORDER BY ORDINAL_POSITION;

-- List all views in the current database
SELECT TABLE_NAME, VIEW_DEFINITION
FROM INFORMATION_SCHEMA.VIEWS
WHERE TABLE_SCHEMA = DATABASE();

-- List all constraints on a table
SELECT CONSTRAINT_NAME, CONSTRAINT_TYPE
FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS
WHERE TABLE_SCHEMA = DATABASE()
  AND TABLE_NAME = 'customer';

-- List all foreign key relationships
SELECT 
    TABLE_NAME AS child_table,
    COLUMN_NAME AS fk_column,
    REFERENCED_TABLE_NAME AS parent_table,
    REFERENCED_COLUMN_NAME AS parent_column,
    CONSTRAINT_NAME
FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE
WHERE REFERENCED_TABLE_NAME IS NOT NULL
  AND TABLE_SCHEMA = DATABASE();
```

### Common INFORMATION_SCHEMA Tables

| Table | What it contains |
|---|---|
| `TABLES` | All tables and views (name, type, engine, row count) |
| `COLUMNS` | All columns (name, data type, nullability, default, ordinal position) |
| `VIEWS` | View definitions |
| `TABLE_CONSTRAINTS` | Constraint names and types (PRIMARY KEY, FOREIGN KEY, UNIQUE, CHECK) |
| `KEY_COLUMN_USAGE` | Which columns are part of a key/constraint, including FK references |
| `STATISTICS` | Index information (index name, columns, uniqueness, cardinality) |
| `ROUTINES` | Stored procedures and functions |
| `SCHEMATA` | Available databases/schemas |
| `USER_PRIVILEGES` | User-level privileges |

```ad-note
`INFORMATION_SCHEMA` is supported by MySQL, MariaDB, PostgreSQL, and SQL Server. The table names and columns are mostly the same across databases, making it the most portable way to query metadata. Oracle uses a different system (`ALL_TABLES`, `ALL_TAB_COLUMNS`, etc.).
```

---

### SHOW Commands — MySQL/MariaDB Shortcut

- MySQL provides `SHOW` commands as a convenient shorthand for common metadata queries. They return the same information as `INFORMATION_SCHEMA` but with simpler syntax:

```sql
-- List all databases
SHOW DATABASES;

-- List all tables in the current database
SHOW TABLES;

-- Show table structure (columns, types, keys)
DESCRIBE customer;
-- or equivalently:
SHOW COLUMNS FROM customer;

-- Show the CREATE statement that would recreate a table
SHOW CREATE TABLE customer;

-- Show the CREATE statement for a view
SHOW CREATE VIEW country_payment_vw;

-- Show all indexes on a table
SHOW INDEX FROM customer;

-- Show available storage engines
SHOW ENGINES;

-- Show current server variables
SHOW VARIABLES LIKE 'innodb%';
```

### SHOW vs INFORMATION_SCHEMA

| | `SHOW` commands | `INFORMATION_SCHEMA` |
|---|---|---|
| **Syntax** | Simple, quick | Standard SQL `SELECT` |
| **Filterable** | Limited (`LIKE`, `WHERE` on some) | Full `WHERE`, `JOIN`, `ORDER BY` |
| **Portable** | MySQL/MariaDB only | SQL standard — works across databases |
| **Joinable** | No — returns a result set you can't join to other tables | Yes — it's a regular `SELECT` |
| **Best for** | Quick interactive exploration | Scripts, dynamic SQL, cross-database compatibility |

```ad-tip
Use `SHOW` when you're exploring interactively in a MySQL terminal. Use `INFORMATION_SCHEMA` when writing scripts or when you need to filter, join, or process metadata programmatically.
```

---

### Practical Examples

```sql
-- Find all tables that have a 'customer_id' column
SELECT TABLE_NAME
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = DATABASE()
  AND COLUMN_NAME = 'customer_id';

-- Find all tables without a primary key
SELECT t.TABLE_NAME
FROM INFORMATION_SCHEMA.TABLES t
LEFT JOIN INFORMATION_SCHEMA.TABLE_CONSTRAINTS c
  ON t.TABLE_NAME = c.TABLE_NAME
  AND t.TABLE_SCHEMA = c.TABLE_SCHEMA
  AND c.CONSTRAINT_TYPE = 'PRIMARY KEY'
WHERE t.TABLE_SCHEMA = DATABASE()
  AND t.TABLE_TYPE = 'BASE TABLE'
  AND c.CONSTRAINT_NAME IS NULL;

-- Count columns per table
SELECT TABLE_NAME, COUNT(*) AS column_count
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = DATABASE()
GROUP BY TABLE_NAME
ORDER BY column_count DESC;
```

```ad-note
`DATABASE()` returns the name of the currently selected database in MySQL. Always filter by `TABLE_SCHEMA = DATABASE()` to avoid seeing metadata from other databases on the same server.
```
