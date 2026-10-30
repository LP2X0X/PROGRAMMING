---
tags: [sql, programmability, dynamic-sql]
---

- **Dynamic SQL** is SQL that is **constructed as a string at runtime** and then executed. Unlike static SQL (where the full query is written at development time), dynamic SQL lets you build queries on the fly — choosing which tables, columns, conditions, or operations to include based on runtime information.
- This is necessary because SQL does not allow variables in structural positions — you cannot write `SELECT * FROM @table_name` or `ORDER BY @column_name`. Dynamic SQL is the workaround.

```sql
-- Static SQL — fully known at development time
SELECT * FROM customer WHERE customer_id = 5;

-- Dynamic SQL — built from a string at runtime
SET @sql = 'SELECT * FROM customer WHERE customer_id = 5';
PREPARE stmt FROM @sql;
EXECUTE stmt;
DEALLOCATE PREPARE stmt;
```

---

### The 3-Step Lifecycle

Dynamic SQL in MySQL uses a **prepare-execute-deallocate** pattern:

```
1. PREPARE    → "Here's a string — parse it as SQL"
2. EXECUTE    → "Run it"
3. DEALLOCATE → "Free the resources"
```

#### Step 1: PREPARE

```sql
SET @sql = 'SELECT * FROM customer WHERE active = 1';
PREPARE stmt FROM @sql;
```

The engine receives the string and:
1. **Parses** it — is it valid SQL syntax?
2. **Security checks** — does the current user have permission to access the referenced tables/columns?
3. **Optimizes** — builds an execution plan (which indexes to use, join order, etc.)

If the string is not valid SQL, PREPARE fails with a syntax error. The query is **not executed** yet — it's only been analyzed and planned.

The name `stmt` is a **statement handle** — a label you use to refer to this prepared statement in the EXECUTE and DEALLOCATE steps. You can choose any name.

#### Step 2: EXECUTE

```sql
EXECUTE stmt;
```

Runs the prepared statement. The engine uses the execution plan created during PREPARE — it does **not** re-parse or re-optimize.

A prepared statement can be executed **multiple times** without re-preparing:

```sql
PREPARE stmt FROM 'SELECT NOW()';
EXECUTE stmt;  -- runs the query
EXECUTE stmt;  -- runs it again (different time, same plan)
EXECUTE stmt;  -- and again
DEALLOCATE PREPARE stmt;
```

This is a performance benefit when running the same query repeatedly with different parameters (see below).

#### Step 3: DEALLOCATE PREPARE

```sql
DEALLOCATE PREPARE stmt;
```

Frees all resources associated with the prepared statement — memory, execution plan, cursors. **Always do this when you're done.** Forgetting to deallocate leaks resources for the duration of the session.

```ad-warning
A prepared statement exists only within the **session** that created it. It cannot be shared across sessions/connections. When the session ends, any prepared statements that were not explicitly deallocated are automatically freed — but relying on this is bad practice. Always `DEALLOCATE PREPARE` explicitly.
```

---

### Parameters with `?` Placeholders

Instead of concatenating values into the SQL string (which is dangerous — see SQL injection below), use `?` as a placeholder and supply the value at EXECUTE time:

```sql
-- One parameter
PREPARE stmt FROM 'SELECT * FROM customer WHERE customer_id = ?';
SET @id = 5;
EXECUTE stmt USING @id;
DEALLOCATE PREPARE stmt;
```

The `?` is a **bind variable**. The engine substitutes `@id` for `?` at execution time. The value is treated as data, never as SQL code — this is what prevents SQL injection.

#### Multiple Parameters

```sql
PREPARE stmt FROM 
    'SELECT * FROM rental WHERE customer_id = ? AND rental_date > ?';
SET @id = 5;
SET @date = '2024-01-01';
EXECUTE stmt USING @id, @date;
DEALLOCATE PREPARE stmt;
```

Parameters are matched **by position** — first `?` gets the first value after `USING`, second `?` gets the second, and so on.

#### Reusing with Different Values

```sql
PREPARE stmt FROM 'SELECT * FROM customer WHERE customer_id = ?';

SET @id = 1;
EXECUTE stmt USING @id;  -- finds customer 1

SET @id = 2;
EXECUTE stmt USING @id;  -- finds customer 2

SET @id = 3;
EXECUTE stmt USING @id;  -- finds customer 3

DEALLOCATE PREPARE stmt;
-- Parsed and optimized once, executed three times
```

```ad-important
`?` placeholders only work for **values** — they cannot replace table names, column names, keywords, or any structural part of the query:

- `WHERE customer_id = ?` — valid (value)
- `SELECT * FROM ?` — **invalid** (table name)
- `ORDER BY ?` — **invalid** (column name)

For structural parts, you must use string concatenation with `CONCAT()`, and you must ensure the values come from a trusted source.
```

---

### Why Use Dynamic SQL?

#### 1. Dynamic Table or Column Names

SQL doesn't allow variables as identifiers. Dynamic SQL is the only way:

```sql
SET @table = 'customer';
SET @sql = CONCAT('SELECT COUNT(*) FROM ', @table);
PREPARE stmt FROM @sql;
EXECUTE stmt;
DEALLOCATE PREPARE stmt;
```

#### 2. Conditional Query Building

Build different queries based on which parameters are provided:

```sql
-- Inside a stored procedure:
SET @sql = 'SELECT * FROM customer WHERE 1=1';

IF p_name IS NOT NULL THEN
    SET @sql = CONCAT(@sql, ' AND last_name = ''', p_name, '''');
END IF;

IF p_active IS NOT NULL THEN
    SET @sql = CONCAT(@sql, ' AND active = ', p_active);
END IF;

IF p_limit IS NOT NULL THEN
    SET @sql = CONCAT(@sql, ' LIMIT ', p_limit);
END IF;

PREPARE stmt FROM @sql;
EXECUTE stmt;
DEALLOCATE PREPARE stmt;
```

The `WHERE 1=1` trick makes it easy to append `AND` clauses without worrying whether it's the first condition or not — `1=1` is always true and has no effect on the result.

#### 3. Metadata-Driven Scripts

Build queries by reading the database's own structure from [[Metadata|INFORMATION_SCHEMA]]:

```sql
-- Dynamically generate a SELECT for all columns in a table
SELECT CONCAT(
    'SELECT ', 
    GROUP_CONCAT(column_name ORDER BY ordinal_position), 
    ' FROM customer'
)
INTO @sql
FROM INFORMATION_SCHEMA.COLUMNS
WHERE table_name = 'customer' AND table_schema = DATABASE();

PREPARE stmt FROM @sql;
EXECUTE stmt;
DEALLOCATE PREPARE stmt;
```

#### 4. Schema Generation

The schema generation pattern from the book — querying `INFORMATION_SCHEMA` to produce `CREATE TABLE` statements:

```sql
-- Build a CREATE TABLE statement from metadata
SELECT CONCAT(
    'CREATE TABLE category (\n',
    GROUP_CONCAT(
        CONCAT('  ', column_name, ' ', column_type,
            CASE WHEN is_nullable = 'NO' THEN ' NOT NULL' ELSE '' END
        )
        ORDER BY ordinal_position
        SEPARATOR ',\n'
    ),
    '\n)'
)
INTO @sql
FROM INFORMATION_SCHEMA.COLUMNS
WHERE table_name = 'category' AND table_schema = DATABASE();

-- @sql now contains a valid CREATE TABLE statement
```

#### 5. Inside Stored Procedures

Dynamic SQL is commonly used inside [[Stored Procedures]] to create reusable, flexible routines:

```sql
DELIMITER //
CREATE PROCEDURE get_table_count(IN tbl_name VARCHAR(64))
BEGIN
    SET @sql = CONCAT('SELECT COUNT(*) AS row_count FROM ', tbl_name);
    PREPARE stmt FROM @sql;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;
END //
DELIMITER ;

-- Call with any table name
CALL get_table_count('customer');   -- row_count: 599
CALL get_table_count('rental');     -- row_count: 16044
CALL get_table_count('payment');    -- row_count: 16049
```

#### 6. Pivot / Dynamic Column Queries

When you don't know how many columns the result will have:

```sql
-- Build a query that pivots category names into columns
SELECT GROUP_CONCAT(
    CONCAT('SUM(CASE WHEN category = ''', name, ''' THEN 1 ELSE 0 END) AS `', name, '`')
)
INTO @cols
FROM category;

SET @sql = CONCAT('SELECT customer_id, ', @cols, ' FROM film_list GROUP BY customer_id');
PREPARE stmt FROM @sql;
EXECUTE stmt;
DEALLOCATE PREPARE stmt;
```

---

### SQL Injection — The Critical Danger

When you concatenate **user-provided input** directly into a SQL string, an attacker can inject their own SQL code:

```sql
-- !! DANGEROUS — never do this with user input !!
SET @user_input = 'Park';
SET @sql = CONCAT('SELECT * FROM customer WHERE last_name = ''', @user_input, '''');
-- Produces: SELECT * FROM customer WHERE last_name = 'Park'
-- Looks fine...

-- But if @user_input = "'; DROP TABLE customer; --"
-- Produces: SELECT * FROM customer WHERE last_name = ''; DROP TABLE customer; --'
-- The attacker just deleted your table!
```

#### The Fix: Use `?` Placeholders for Values

```sql
-- SAFE — the value is never part of the SQL string
PREPARE stmt FROM 'SELECT * FROM customer WHERE last_name = ?';
SET @name = @user_input;  -- even if this contains malicious SQL
EXECUTE stmt USING @name; -- it's treated as a data value, not SQL code
DEALLOCATE PREPARE stmt;
```

The `?` placeholder ensures the value is **always treated as data**, no matter what it contains. The engine never parses it as SQL.

#### When You Can't Use `?`

For table names and column names, `?` doesn't work. In these cases, **validate against a whitelist**:

```sql
-- Inside a stored procedure
IF tbl_name NOT IN ('customer', 'rental', 'payment', 'film') THEN
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Invalid table name';
END IF;

SET @sql = CONCAT('SELECT * FROM ', tbl_name);
-- Safe because tbl_name was validated against a known list
```

```ad-warning
**The Rule:**
- For **values** → always use `?` placeholders
- For **identifiers** (table/column names) → validate against a whitelist of known names, never accept raw user input
- **Never** concatenate user input directly into a SQL string
```

---

### Limitations and Trade-offs

| Limitation | Detail |
|---|---|
| **Can't parameterize identifiers** | `?` only works for values. Table names, column names, and keywords must be concatenated |
| **Session-scoped** | A prepared statement only exists in the session that created it — not shared across connections |
| **Must deallocate** | Forgetting `DEALLOCATE PREPARE` leaks resources until the session ends |
| **Harder to debug** | The query is a string — typos, missing quotes, and syntax errors become runtime errors that are harder to trace |
| **No compile-time checking** | Static SQL is checked when the procedure is created. Dynamic SQL is only checked when PREPARE runs — bugs hide until execution |
| **SQL injection risk** | If you concatenate user input instead of using `?`, you open a critical security vulnerability |
| **Optimizer limitations** | The engine optimizes at PREPARE time. If data distribution changes significantly between PREPARE and EXECUTE, the plan may be suboptimal |

### When to Use vs When to Avoid

| Use dynamic SQL when... | Avoid dynamic SQL when... |
|---|---|
| Table or column names are determined at runtime | The query is fully known at development time |
| Building flexible search with optional conditions | A static query with `CASE` or `COALESCE` can handle it |
| Generating schema scripts from metadata | You can use `mysqldump` or similar tools |
| Creating reusable procedures that work across tables | The procedure only works with one specific table |
| Pivoting with a dynamic number of columns | The columns are known and fixed |

```ad-tip
Dynamic SQL is powerful but adds complexity. Prefer static SQL when possible — it's safer, easier to debug, and catches errors earlier. Only reach for dynamic SQL when the query's **structure** (not just its values) needs to change at runtime.
```

---

### C# Analogy

Dynamic SQL is like **reflection** in C#:

```csharp
// Static — fully known at compile time
var customer = new Customer();
customer.Name = "Park";

// Dynamic — built at runtime using reflection
Type type = Type.GetType("Customer");
object obj = Activator.CreateInstance(type);
type.GetProperty("Name").SetValue(obj, "Park");
```

Same trade-offs: more flexibility, less safety, harder to debug, and you lose compile-time checking.

The `?` placeholder concept maps directly to **parameterized queries** in ADO.NET:

```csharp
// C# equivalent of PREPARE with ?
var cmd = new MySqlCommand(
    "SELECT * FROM customer WHERE customer_id = @id", connection);
cmd.Parameters.AddWithValue("@id", 5);
cmd.ExecuteReader();
// @id is always treated as data, never as SQL — same protection as ?
```

Both prevent SQL injection by separating the query structure from its data values.
