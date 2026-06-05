---
tags: [database, design, constraints]
---

## What Are Constraints?

- **Constraints** are rules defined at the database level that enforce **data integrity** — they guarantee that the data stored in your tables is valid, consistent, and meaningful.
- Constraints act as the **last line of defense** for your data. Even if your application code has bugs, even if someone runs raw SQL directly against the database, even if a new developer forgets a validation rule — the constraints will reject invalid data and protect the integrity of your database.
- Every constraint answers a simple question: "What must always be true about this data?" The database engine checks the answer on every `INSERT`, `UPDATE`, and (for some constraints) `DELETE`, and rejects the operation if the rule would be violated.

```ad-important
Never rely solely on application-level validation. Application code can have bugs, can be bypassed through direct SQL access, and may not be consistent across multiple applications hitting the same database. Constraints are enforced by the DBMS itself — they cannot be bypassed, they cannot be forgotten, and they cannot have bugs. ==Always define constraints at the database level, even if you also validate in the application.==
```

---

## Constraint Types — Overview

| Constraint | What It Enforces | Scope |
| --- | --- | --- |
| **PRIMARY KEY** | Each row is uniquely identifiable; the key column(s) cannot be NULL | One per table |
| **FOREIGN KEY** | A value must exist in the referenced parent table | Many per table |
| **UNIQUE** | No duplicate values in the column(s); allows one NULL | Many per table |
| **NOT NULL** | The column must have a value — NULL is not allowed | Per column |
| **CHECK** | A custom Boolean condition must evaluate to TRUE | Many per table |
| **DEFAULT** | Provides an automatic value when none is supplied during INSERT | Per column |

---

## PRIMARY KEY Constraint

- The **primary key** uniquely identifies each row in a table. It combines two rules: the value must be **unique** across all rows, and it must be **NOT NULL**.
- Each table can have **exactly one** primary key, but the key can span multiple columns (a **composite primary key**).
- The primary key also creates a **clustered [[02 - Indexes|index]]** by default (in SQL Server and MySQL/InnoDB), which determines the physical sort order of the data on disk.

```sql
-- Single-column primary key
CREATE TABLE Users (
    Id    INT PRIMARY KEY IDENTITY,  -- IDENTITY = auto-increment (SQL Server)
    Name  VARCHAR(100),
    Email VARCHAR(200)
);

-- Composite primary key (for junction/bridge tables)
CREATE TABLE Enrollments (
    StudentId INT,
    CourseId  INT,
    Grade     CHAR(1),
    PRIMARY KEY (StudentId, CourseId)  -- combination must be unique
);
```

### Natural Key vs Surrogate Key

| | Natural Key | Surrogate Key |
| --- | --- | --- |
| **What** | A column with real-world meaning (SSN, Email, ISBN) | An artificial column with no business meaning (auto-increment Id) |
| **Pros** | Meaningful, no extra column needed | Stable (never changes), compact, simple |
| **Cons** | Can change (email changes, SSN is reassigned), may be long, may have format issues | Meaningless on its own, requires lookup for business context |
| **Recommendation** | Use as UNIQUE constraint, not PK | ==Use surrogate keys as your primary key in most cases== |

```ad-note
The surrogate key approach (`Id INT PRIMARY KEY IDENTITY`) is the industry standard for application databases. Natural keys as primary keys cause problems when the business value changes — every foreign key referencing that PK would also need to change, cascading across the entire schema. Keep natural keys as UNIQUE constraints for data integrity, but let the PK be an immutable integer.
```

---

## FOREIGN KEY Constraint

- A **foreign key** enforces **referential integrity** — it guarantees that a value in one table must exist in another table's referenced column (typically the primary key or a unique column).
- Foreign keys define [[relationships]] between tables. They are the structural mechanism that makes the relational model work.

```sql
CREATE TABLE Customers (
    Id    INT PRIMARY KEY IDENTITY,
    Name  VARCHAR(100) NOT NULL,
    Email VARCHAR(200) NOT NULL
);

CREATE TABLE Orders (
    Id         INT PRIMARY KEY IDENTITY,
    CustomerId INT NOT NULL,
    OrderDate  DATETIME2 DEFAULT GETDATE(),
    -- Foreign key: CustomerId must exist in Customers.Id
    CONSTRAINT FK_Orders_CustomerId
        FOREIGN KEY (CustomerId) REFERENCES Customers(Id)
);
```

- With this constraint in place:
  - You **cannot** insert an order with a `CustomerId` that does not exist in the `Customers` table.
  - You **cannot** delete a customer who has existing orders (by default).
  - The database guarantees there are no "orphan" orders pointing to nonexistent customers.

### Referential Actions: ON DELETE and ON UPDATE

- When a referenced row in the parent table is deleted or updated, the foreign key constraint determines what happens to the dependent rows in the child table:

| Action | ON DELETE behavior | ON UPDATE behavior |
| --- | --- | --- |
| **NO ACTION** (default) | Prevent deletion if child rows exist | Prevent update if child rows reference the old value |
| **CASCADE** | Delete all child rows when the parent row is deleted | Update all child rows with the new parent value |
| **SET NULL** | Set the FK column to NULL in child rows | Set the FK column to NULL in child rows |
| **SET DEFAULT** | Set the FK column to its DEFAULT value | Set the FK column to its DEFAULT value |

```sql
-- CASCADE: deleting a customer also deletes all their orders
CONSTRAINT FK_Orders_CustomerId
    FOREIGN KEY (CustomerId) REFERENCES Customers(Id)
    ON DELETE CASCADE;

-- SET NULL: deleting a department sets employees' DeptId to NULL
CONSTRAINT FK_Employees_DeptId
    FOREIGN KEY (DepartmentId) REFERENCES Departments(Id)
    ON DELETE SET NULL;
```

```ad-warning
Use `ON DELETE CASCADE` with extreme caution. It means deleting one row in the parent table can silently delete thousands of rows in child tables — and if those child tables also have cascading FKs, the deletion can cascade through the entire database. This is rarely the desired behavior for most business data. The safe default is `NO ACTION` — it forces you to explicitly handle dependent rows before deleting a parent.
```

```ad-note
Always [[02 - Indexes|index]] your foreign key columns. Without an index, the database must perform a full table scan on the child table every time it needs to check or enforce the foreign key relationship — this happens on every join, parent delete, and parent update. Most databases do not automatically create an index on foreign key columns (SQL Server does not; MySQL/InnoDB does).
```

---

## UNIQUE Constraint

- A **UNIQUE** constraint ensures that no two rows have the same value in the specified column(s). Unlike PRIMARY KEY, a UNIQUE constraint **allows one NULL** value (in most databases — the reasoning is that NULL means "unknown", and you cannot say two unknown values are the same).
- A table can have **many** UNIQUE constraints. Behind the scenes, each UNIQUE constraint creates a unique non-clustered [[02 - Indexes|index]].

```sql
CREATE TABLE Users (
    Id    INT PRIMARY KEY IDENTITY,
    Email VARCHAR(200) NOT NULL UNIQUE,  -- no two users can have the same email
    Phone VARCHAR(20) UNIQUE             -- nullable, but if provided, must be unique
);

-- Named UNIQUE constraint (better for debugging and migration scripts)
CREATE TABLE Users (
    Id    INT PRIMARY KEY IDENTITY,
    Email VARCHAR(200) NOT NULL,
    Phone VARCHAR(20),
    CONSTRAINT UQ_Users_Email UNIQUE (Email),
    CONSTRAINT UQ_Users_Phone UNIQUE (Phone)
);

-- Composite UNIQUE: the combination must be unique
CREATE TABLE CourseSchedule (
    Id        INT PRIMARY KEY IDENTITY,
    RoomId    INT NOT NULL,
    TimeSlot  DATETIME2 NOT NULL,
    CONSTRAINT UQ_Schedule_Room_Time UNIQUE (RoomId, TimeSlot)
    -- Same room can't have two classes at the same time
);
```

### UNIQUE vs PRIMARY KEY

| | PRIMARY KEY | UNIQUE |
| --- | --- | --- |
| NULLs allowed? | No | Yes (one NULL in most databases) |
| How many per table? | Exactly one | Many |
| Index type created | Clustered (by default) | Non-clustered |
| Purpose | Row identity | Business rule enforcement |

---

## NOT NULL Constraint

- **NOT NULL** ensures a column must always have a value — `NULL` is not permitted on `INSERT` or `UPDATE`.
- This is the simplest constraint, but also one of the most important. A `NULL` in a column where you expect data silently corrupts your business logic — aggregations skip NULLs, comparisons with NULL return UNKNOWN, and joins on NULL never match.

```sql
CREATE TABLE Products (
    Id    INT PRIMARY KEY IDENTITY,
    Name  VARCHAR(100) NOT NULL,   -- every product MUST have a name
    Price DECIMAL(10,2) NOT NULL,  -- every product MUST have a price
    Description VARCHAR(500)       -- description is optional (NULL allowed)
);
```

```ad-note
Be deliberate about which columns allow NULL. The default in SQL is that columns *do* allow NULL unless you specify `NOT NULL`. This means if you forget to add `NOT NULL`, the column silently accepts missing data. A good practice: **start by making every column NOT NULL**, then only allow NULL where you have a clear business reason for "no value" (e.g., an optional phone number, a nullable foreign key for an optional relationship).
```

### NULL Behavior Gotchas

```sql
-- NULL comparisons are always UNKNOWN (not TRUE, not FALSE)
SELECT * FROM Users WHERE Phone = NULL;       -- Returns NOTHING (wrong!)
SELECT * FROM Users WHERE Phone IS NULL;      -- Correct way to check for NULL

-- NULL in arithmetic propagates
SELECT Price * NULL FROM Products;            -- Returns NULL

-- Aggregates skip NULLs
SELECT AVG(Rating) FROM Reviews;
-- If ratings are [5, NULL, 3], AVG = (5+3)/2 = 4.0, not (5+0+3)/3 = 2.67
-- NULL is skipped entirely, not treated as 0
```

---

## CHECK Constraint

- A **CHECK** constraint enforces a custom Boolean condition on column values. The database rejects any `INSERT` or `UPDATE` where the condition evaluates to FALSE.
- CHECK constraints can reference multiple columns in the same row, enabling cross-column validation.

```sql
CREATE TABLE Products (
    Id       INT PRIMARY KEY IDENTITY,
    Name     VARCHAR(100) NOT NULL,
    Price    DECIMAL(10,2) NOT NULL,
    Quantity INT NOT NULL,
    Category VARCHAR(50) NOT NULL,

    -- Price must be positive
    CONSTRAINT CK_Products_Price CHECK (Price > 0),

    -- Quantity cannot be negative
    CONSTRAINT CK_Products_Quantity CHECK (Quantity >= 0),

    -- Category must be one of the allowed values
    CONSTRAINT CK_Products_Category CHECK (Category IN ('Electronics', 'Books', 'Clothing', 'Food'))
);
```

### Multi-Column CHECK Constraints

```sql
CREATE TABLE Events (
    Id        INT PRIMARY KEY IDENTITY,
    StartDate DATETIME2 NOT NULL,
    EndDate   DATETIME2 NOT NULL,

    -- End date must be after start date
    CONSTRAINT CK_Events_DateRange CHECK (EndDate > StartDate)
);

CREATE TABLE Discounts (
    Id              INT PRIMARY KEY IDENTITY,
    DiscountPercent DECIMAL(5,2),
    DiscountAmount  DECIMAL(10,2),

    -- Must have exactly one type of discount, not both
    CONSTRAINT CK_Discounts_OneType CHECK (
        (DiscountPercent IS NOT NULL AND DiscountAmount IS NULL)
        OR
        (DiscountPercent IS NULL AND DiscountAmount IS NOT NULL)
    )
);
```

```ad-warning
CHECK constraints have a subtle behavior with NULLs: a CHECK constraint passes if the condition evaluates to TRUE **or UNKNOWN (NULL)**. This means `CHECK (Price > 0)` allows NULL prices — because `NULL > 0` evaluates to UNKNOWN, not FALSE. If you want to prevent NULLs, you must also add a `NOT NULL` constraint. The CHECK and NOT NULL constraints work together, not as substitutes for each other.
```

### CHECK vs Lookup Table

- For restricting a column to a set of allowed values, you have two options:

| Approach | Pros | Cons |
| --- | --- | --- |
| `CHECK (Category IN (...))` | Simple, self-contained, no extra table | Hard-coded — adding a new category requires `ALTER TABLE` |
| Lookup table with FOREIGN KEY | New values added with `INSERT`, no schema change | Extra table, extra join |

- **Rule of thumb:** If the set of values is small and rarely changes (e.g., Status: Active/Inactive, Gender: M/F/Other), a CHECK constraint is fine. If the set of values changes over time or is managed by users (e.g., product categories, country codes), use a [[lookup table]] with a foreign key. See [[04 - Schema Design Patterns]] for the lookup table pattern.

---

## DEFAULT Constraint

- A **DEFAULT** constraint provides a value for a column when no value is specified during `INSERT`. It does not prevent explicit NULLs (if the column allows NULL) — it only fills in when the column is *omitted* from the INSERT statement.

```sql
CREATE TABLE Orders (
    Id        INT PRIMARY KEY IDENTITY,
    OrderDate DATETIME2   DEFAULT GETDATE(),      -- auto-fills with current timestamp
    Status    VARCHAR(20) DEFAULT 'Pending',      -- new orders start as Pending
    IsActive  BIT         DEFAULT 1,              -- active by default
    Notes     VARCHAR(500) DEFAULT ''             -- empty string, not NULL
);

-- These INSERTs all use defaults:
INSERT INTO Orders DEFAULT VALUES;
-- OrderDate = now, Status = 'Pending', IsActive = 1, Notes = ''

INSERT INTO Orders (Status) VALUES ('Shipped');
-- OrderDate = now (default), Status = 'Shipped' (explicit), IsActive = 1 (default)
```

### Common DEFAULT Values

| Column Type | Common Default |
| --- | --- |
| Timestamps | `GETDATE()` / `CURRENT_TIMESTAMP` / `NOW()` |
| Status flags | `'Pending'`, `'Draft'`, `'Active'` |
| Boolean flags | `1` (true) / `0` (false) |
| Counters | `0` |
| Strings | `''` (empty string) |
| GUIDs | `NEWID()` (SQL Server) / `UUID()` |

```ad-note
DEFAULT and NOT NULL work well together. A column that is `NOT NULL DEFAULT 'Pending'` means: the column always has a value — if you provide one, it uses yours; if you don't, it uses 'Pending'. This combination is the standard pattern for status columns, audit timestamps, and boolean flags.
```

---

## Naming Conventions for Constraints

- Giving constraints meaningful names is critical for debugging and maintenance. When a constraint violation occurs, the error message includes the constraint name. A good name tells you exactly what failed; a system-generated name like `CK__Products__Price__2A4B4B5E` tells you nothing.

| Constraint Type | Naming Convention | Example |
| --- | --- | --- |
| Primary Key | `PK_TableName` | `PK_Users` |
| Foreign Key | `FK_ChildTable_ColumnName` | `FK_Orders_CustomerId` |
| Unique | `UQ_TableName_ColumnName` | `UQ_Users_Email` |
| Check | `CK_TableName_Rule` | `CK_Products_Price` |
| Default | `DF_TableName_ColumnName` | `DF_Orders_Status` |

```sql
CREATE TABLE Products (
    Id       INT NOT NULL,
    Name     VARCHAR(100) NOT NULL,
    Price    DECIMAL(10,2) NOT NULL,
    Category VARCHAR(50) NOT NULL,

    CONSTRAINT PK_Products PRIMARY KEY (Id),
    CONSTRAINT CK_Products_Price CHECK (Price > 0),
    CONSTRAINT CK_Products_Category CHECK (Category IN ('Electronics', 'Books', 'Clothing'))
);
```

```ad-note
Always name your constraints explicitly, even though most databases will auto-generate a name if you don't. Auto-generated names are inconsistent across environments — a constraint created in development may have a different auto-generated name in production, making migration scripts and error handling unreliable. Explicit names give you control and clarity.
```

---

## Adding and Removing Constraints on Existing Tables

- Constraints can be added to or removed from existing tables using `ALTER TABLE`. This is how you evolve your schema over time.

### Adding Constraints

```sql
-- Add a CHECK constraint
ALTER TABLE Products
ADD CONSTRAINT CK_Products_Price CHECK (Price > 0);

-- Add a UNIQUE constraint
ALTER TABLE Users
ADD CONSTRAINT UQ_Users_Email UNIQUE (Email);

-- Add a FOREIGN KEY constraint
ALTER TABLE Orders
ADD CONSTRAINT FK_Orders_CustomerId
    FOREIGN KEY (CustomerId) REFERENCES Customers(Id);

-- Add a DEFAULT constraint
ALTER TABLE Orders
ADD CONSTRAINT DF_Orders_Status DEFAULT 'Pending' FOR Status;

-- Add NOT NULL (syntax varies — SQL Server uses ALTER COLUMN)
ALTER TABLE Products
ALTER COLUMN Name VARCHAR(100) NOT NULL;
```

```ad-warning
When adding a constraint to an existing table, the database will check all existing data against the new constraint. If any existing rows violate the constraint, the `ALTER TABLE` will fail. You must clean up the violating data first (or use `WITH NOCHECK` in SQL Server to skip validation of existing data — but this is generally a bad practice because it means your constraint is not truly enforced for all rows).
```

### Removing Constraints

```sql
-- Drop a named constraint
ALTER TABLE Products
DROP CONSTRAINT CK_Products_Price;

ALTER TABLE Users
DROP CONSTRAINT UQ_Users_Email;

ALTER TABLE Orders
DROP CONSTRAINT FK_Orders_CustomerId;

-- Drop NOT NULL (SQL Server — change column to allow NULL)
ALTER TABLE Products
ALTER COLUMN Name VARCHAR(100) NULL;
```

---

## Constraints vs Application Validation

- A common question: "If I validate in my C# code, do I still need database constraints?" The answer is **yes, always both**.

| | Database Constraints | Application Validation |
| --- | --- | --- |
| **Where** | Database engine (DBMS) | Application code (C#, etc.) |
| **Bypassable?** | No — enforced by the engine regardless of how data enters | Yes — bugs, direct SQL access, other apps hitting the same DB |
| **User feedback** | Generic error message ("CHECK constraint violation") | Friendly UI messages ("Price must be greater than zero") |
| **Performance** | Extremely fast — checked at the engine level | Varies — may involve network round-trips for uniqueness checks |
| **Scope** | Protects data integrity against ALL writers | Protects only one application's writes |

### The Defense-in-Depth Model

```
User Input
    │
    ▼
Application Validation (C# / .NET)
    │  ← Catches most errors early, provides friendly messages
    │  ← Can do complex validation (business rules, async checks)
    ▼
Database Constraints
    │  ← Catches anything the application missed
    │  ← Cannot be bypassed — final guarantee of data integrity
    ▼
Data Stored ✓
```

- **Application validation** provides a good user experience — it catches errors early and returns helpful, specific error messages before the data ever reaches the database.
- **Database constraints** provide data integrity — they are the safety net that catches anything the application missed, including bugs in the application, direct SQL access, and other applications writing to the same database.

```ad-important
Think of it this way: application validation is for **user experience**. Database constraints are for **data correctness**. You need both. The application makes the user's life easier. The constraints make the data's life safe.
```

---

## Common Constraint Patterns in Practice

### The Complete Table Example

```sql
CREATE TABLE Products (
    -- Primary key: identity, unique, not null
    Id          INT NOT NULL IDENTITY,
    CONSTRAINT PK_Products PRIMARY KEY (Id),

    -- Required fields
    Name        VARCHAR(100) NOT NULL,
    SKU         VARCHAR(50) NOT NULL,
    Price       DECIMAL(10,2) NOT NULL,
    Quantity    INT NOT NULL,
    CategoryId  INT NOT NULL,

    -- Optional fields
    Description VARCHAR(500) NULL,
    ImageUrl    VARCHAR(500) NULL,

    -- Audit columns with defaults
    CreatedAt   DATETIME2 NOT NULL DEFAULT GETDATE(),
    IsActive    BIT NOT NULL DEFAULT 1,

    -- Business rules
    CONSTRAINT UQ_Products_SKU UNIQUE (SKU),
    CONSTRAINT CK_Products_Price CHECK (Price > 0),
    CONSTRAINT CK_Products_Quantity CHECK (Quantity >= 0),
    CONSTRAINT FK_Products_CategoryId
        FOREIGN KEY (CategoryId) REFERENCES Categories(Id)
);
```

- This table demonstrates the constraint philosophy in action:
  - **PK** ensures row identity
  - **NOT NULL** on required columns prevents missing data
  - **UNIQUE** on SKU prevents duplicate products
  - **CHECK** on Price and Quantity enforces business rules
  - **DEFAULT** on CreatedAt and IsActive provides sensible auto-fill values
  - **FK** on CategoryId ensures referential integrity with the Categories table

---

**Next:** [[04 - Schema Design Patterns]]
