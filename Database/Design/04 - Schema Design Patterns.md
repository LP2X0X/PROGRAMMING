---
tags: [database, design, patterns]
---

## Overview

- This note covers practical, real-world patterns and conventions for designing database schemas. The previous notes covered the theoretical foundations — [[01 - Normalization|normalization]], [[02 - Indexes|indexes]], and [[03 - Constraints|constraints]]. This note ties them together into actionable patterns you will use in nearly every database you design.
- A well-designed schema is not just correct — it is *readable*, *predictable*, and *maintainable*. When another developer (or you, six months from now) looks at the schema, the structure should communicate its purpose clearly through consistent naming, sensible defaults, and recognizable patterns.

---

## Naming Conventions

- Consistent naming is one of the most impactful things you can do for schema maintainability. There is no single "correct" convention — what matters is that you **pick one and follow it everywhere**.

### Table Names

| Convention | Example | Notes |
| --- | --- | --- |
| PascalCase singular | `User`, `Order`, `Product` | Treats the table as a *type* definition |
| PascalCase plural | `Users`, `Orders`, `Products` | Treats the table as a *collection* of rows |

- Both are widely used. **Pick one for your project and be absolutely consistent.** Mixing `User` and `Orders` in the same database is worse than either convention used consistently.

```ad-note
SQL Server and Microsoft conventions tend toward PascalCase singular. MySQL/MariaDB communities often use plural or snake_case. The examples in this note series use PascalCase singular for table names and PascalCase for columns, consistent with C#/.NET conventions — which makes ORM mapping more natural when your entity class `User` maps to a table `Users` or `User`.
```

### Column Names

| Element | Convention | Example |
| --- | --- | --- |
| Regular columns | PascalCase | `FirstName`, `OrderDate`, `IsActive` |
| Primary key | `Id` or `TableNameId` | `Id` or `UserId` |
| Foreign key | `ReferencedTableId` | `CustomerId`, `ProductId`, `CategoryId` |
| Boolean columns | `Is`/`Has`/`Can` prefix | `IsActive`, `IsDeleted`, `HasDiscount`, `CanEdit` |
| Date/time columns | Descriptive suffix | `CreatedAt`, `UpdatedAt`, `DeletedAt`, `OrderDate`, `ShippedOn` |

### Database Object Names

| Object | Prefix Convention | Example |
| --- | --- | --- |
| Index | `IX_` | `IX_Users_Email`, `IX_Orders_CustomerId_Date` |
| Primary Key | `PK_` | `PK_Users`, `PK_Orders` |
| Foreign Key | `FK_` | `FK_Orders_CustomerId` |
| Unique Constraint | `UQ_` | `UQ_Users_Email` |
| Check Constraint | `CK_` | `CK_Products_Price` |
| Default Constraint | `DF_` | `DF_Orders_Status` |
| Stored Procedure | `usp_` | `usp_GetUserById`, `usp_CreateOrder` |
| View | `vw_` | `vw_ActiveUsers`, `vw_OrderSummary` |
| Trigger | `TR_` | `TR_Users_AfterInsert` |

```ad-warning
Avoid using reserved SQL keywords as table or column names. `Order`, `User`, `Group`, `Index`, `Key`, `Table`, `Column`, `Date`, and `Status` are all reserved or semi-reserved in various SQL dialects. If you must use them, you'll need to quote them everywhere (`[Order]` in SQL Server, `` `Order` `` in MySQL), which is error-prone. Better to use `Orders`, `Users`, `Groups`, or add a suffix like `OrderDate`, `UserStatus`.
```

---

## Soft Delete Pattern

- **Soft delete** means marking a row as "deleted" without physically removing it from the database. The row remains in the table but is filtered out of normal queries.
- This pattern is used when you need to:
  - **Preserve data for auditing or compliance** — regulatory requirements may demand that records are never truly deleted
  - **Allow undo/restore** — users can "undelete" records
  - **Maintain referential integrity** — deleting a customer who has orders would violate foreign keys or require cascading deletes; soft delete avoids this entirely
  - **Keep historical data** — analytics and reporting may need access to "deleted" records

### Implementation

```sql
-- Add soft delete columns to a table
ALTER TABLE Users ADD IsDeleted BIT NOT NULL DEFAULT 0;
ALTER TABLE Users ADD DeletedAt DATETIME2 NULL;
ALTER TABLE Users ADD DeletedBy VARCHAR(100) NULL;
```

```sql
-- "Delete" a user (soft delete)
UPDATE Users
SET IsDeleted = 1,
    DeletedAt = GETDATE(),
    DeletedBy = 'admin@company.com'
WHERE Id = 42;

-- Query only active users (the standard query pattern)
SELECT * FROM Users WHERE IsDeleted = 0;

-- Restore a soft-deleted user
UPDATE Users
SET IsDeleted = 0,
    DeletedAt = NULL,
    DeletedBy = NULL
WHERE Id = 42;
```

### Making Soft Delete Transparent with a View

```sql
-- Create a view that hides deleted records
CREATE VIEW vw_ActiveUsers AS
SELECT Id, Name, Email, CreatedAt
FROM Users
WHERE IsDeleted = 0;

-- Now queries against the view automatically exclude soft-deleted rows
SELECT * FROM vw_ActiveUsers;
```

### UNIQUE Constraint Gotcha with Soft Delete

```sql
-- Problem: UNIQUE constraint on Email prevents reuse of soft-deleted emails
-- User A has email "alice@email.com" and is soft-deleted
-- User B tries to register with "alice@email.com" — UNIQUE violation!

-- Solution (SQL Server): filtered unique index — only enforce uniqueness among active rows
CREATE UNIQUE INDEX IX_Users_Email_Active
ON Users(Email)
WHERE IsDeleted = 0;
-- Now the email is only unique among non-deleted users
```

```ad-note
Soft delete adds complexity. Every query must remember to filter `WHERE IsDeleted = 0`, and forgetting this filter leaks "deleted" data to users. Views can mitigate this, but not all ORMs use views seamlessly. Evaluate whether you truly need soft delete — if not, a hard delete with an archive/history table may be simpler and safer.
```

---

## Audit Columns Pattern

- **Audit columns** track *when* a row was created or last modified, and *who* made the change. This is the minimum-viable audit trail that every production table should have.

### Basic Audit Columns

```sql
CREATE TABLE Products (
    Id         INT PRIMARY KEY IDENTITY,
    Name       VARCHAR(100) NOT NULL,
    Price      DECIMAL(10,2) NOT NULL,

    -- Audit columns
    CreatedAt  DATETIME2 NOT NULL DEFAULT GETDATE(),
    CreatedBy  VARCHAR(100) NOT NULL,
    UpdatedAt  DATETIME2 NULL,
    UpdatedBy  VARCHAR(100) NULL
);
```

| Column | Purpose | Default | Nullable? |
| --- | --- | --- | --- |
| `CreatedAt` | When the row was first inserted | `GETDATE()` | No |
| `CreatedBy` | Who inserted the row | None — must be provided | No |
| `UpdatedAt` | When the row was last modified | NULL (never updated yet) | Yes |
| `UpdatedBy` | Who last modified the row | NULL | Yes |

### Keeping Audit Columns Updated

- `CreatedAt` and `CreatedBy` are set once, at insert time — they never change.
- `UpdatedAt` and `UpdatedBy` must be set on every `UPDATE`. You can do this in application code, or use a trigger:

```sql
-- Option 1: Application code (C# / .NET)
-- Every UPDATE statement explicitly sets UpdatedAt and UpdatedBy

-- Option 2: Database trigger (automatic, but less transparent)
CREATE TRIGGER TR_Products_AfterUpdate
ON Products
AFTER UPDATE
AS
BEGIN
    UPDATE Products
    SET UpdatedAt = GETDATE()
    FROM Products p
    INNER JOIN inserted i ON p.Id = i.Id;
END;
```

```ad-note
Triggers are powerful but hidden — they execute automatically and can surprise developers who don't know they exist. If you use triggers for audit columns, document them clearly. Many teams prefer handling audit columns in the application layer (e.g., EF Core's `SaveChanges` override) for transparency and testability.
```

### Full Audit/History Table Pattern

- For sensitive or regulated data, basic audit columns are not enough — you need a full history of every change. The **history table** pattern records every version of a row:

```sql
CREATE TABLE ProductHistory (
    HistoryId   INT PRIMARY KEY IDENTITY,
    ProductId   INT NOT NULL,           -- which product changed
    Name        VARCHAR(100),
    Price       DECIMAL(10,2),
    ChangedAt   DATETIME2 NOT NULL DEFAULT GETDATE(),
    ChangedBy   VARCHAR(100) NOT NULL,
    ChangeType  VARCHAR(10) NOT NULL    -- 'INSERT', 'UPDATE', 'DELETE'
);
```

- A trigger on the Products table copies the old (or new) row into ProductHistory on every `INSERT`, `UPDATE`, and `DELETE`. This creates a complete, immutable audit trail.

```ad-note
SQL Server 2016+ has a built-in feature called **Temporal Tables** (System-Versioned Tables) that automates this pattern. The database engine automatically maintains a history table with row versioning, and you can query "what did this row look like at 3:00 PM yesterday?" using `FOR SYSTEM_TIME AS OF`. If you are on SQL Server 2016+, prefer temporal tables over manual history tables.
```

---

## Lookup / Reference Table Pattern

- **Lookup tables** (also called reference tables or code tables) replace hard-coded string values with foreign key references to a dedicated table of valid values.

### The Problem: Magic Strings

```sql
-- BAD: Status is a free-text column — nothing stops typos
CREATE TABLE Orders (
    Id     INT PRIMARY KEY IDENTITY,
    Status VARCHAR(20)  -- allows 'Pending', 'Pnding', 'pending', 'PENDING', ''
);

INSERT INTO Orders (Status) VALUES ('Pnding');  -- Oops — no error!
```

### The Solution: Lookup Table

```sql
-- GOOD: Separate table defines the valid values
CREATE TABLE OrderStatuses (
    Id   INT PRIMARY KEY,
    Name VARCHAR(50) NOT NULL,
    CONSTRAINT UQ_OrderStatuses_Name UNIQUE (Name)
);

INSERT INTO OrderStatuses (Id, Name) VALUES
    (1, 'Pending'),
    (2, 'Processing'),
    (3, 'Shipped'),
    (4, 'Delivered'),
    (5, 'Cancelled');

CREATE TABLE Orders (
    Id       INT PRIMARY KEY IDENTITY,
    StatusId INT NOT NULL DEFAULT 1,  -- defaults to 'Pending'
    CONSTRAINT FK_Orders_StatusId
        FOREIGN KEY (StatusId) REFERENCES OrderStatuses(Id)
);
```

### Benefits Over CHECK Constraints or Magic Strings

| Feature | Magic String | CHECK Constraint | Lookup Table |
| --- | --- | --- | --- |
| Prevents invalid values | No | Yes | Yes |
| Adding a new value | Just type it (dangerous) | `ALTER TABLE` (requires DDL permissions) | `INSERT` (data-only change) |
| Can store metadata about each value | No | No | Yes — add columns like `SortOrder`, `IsActive`, `Description` |
| Provides consistent display names | No — depends on what was typed | Partially — constraint defines valid strings | Yes — single source of truth |
| Reusable across tables | No | No (constraint is per-table) | Yes — multiple tables can FK to the same lookup |

```sql
-- Lookup table with extra metadata
CREATE TABLE OrderStatuses (
    Id          INT PRIMARY KEY,
    Name        VARCHAR(50) NOT NULL UNIQUE,
    Description VARCHAR(200),
    SortOrder   INT NOT NULL DEFAULT 0,    -- controls display order in UI
    IsActive    BIT NOT NULL DEFAULT 1     -- soft-disable a status without deleting it
);
```

```ad-note
For truly fixed, small sets of values that will never change (e.g., Boolean-like states: Active/Inactive), a [[03 - Constraints|CHECK constraint]] is simpler and sufficient. Use lookup tables when the set of values may change over time, when you need metadata about each value, or when the values are referenced from multiple tables.
```

---

## Self-Referencing Table Pattern

- A **self-referencing table** has a foreign key that points back to its own primary key. This models **hierarchical / tree** structures within a single table — any data where an entity can be a "child" of another entity of the same type.

### Common Use Cases

- **Organizational hierarchy** — employees and their managers
- **Category trees** — a category can have sub-categories
- **Comment threads** — a reply is a child of another comment
- **File system / folder structure** — folders contain sub-folders
- **Bill of materials** — an assembly contains sub-assemblies

### Implementation

```sql
CREATE TABLE Employees (
    Id        INT PRIMARY KEY IDENTITY,
    Name      VARCHAR(100) NOT NULL,
    Title     VARCHAR(100),
    ManagerId INT NULL,  -- NULL = top of the hierarchy (CEO, etc.)
    CONSTRAINT FK_Employees_ManagerId
        FOREIGN KEY (ManagerId) REFERENCES Employees(Id)
);

INSERT INTO Employees (Id, Name, Title, ManagerId) VALUES
    (1, 'Alice',   'CEO',             NULL),  -- top level
    (2, 'Bob',     'VP Engineering',  1),     -- reports to Alice
    (3, 'Charlie', 'VP Sales',        1),     -- reports to Alice
    (4, 'Dave',    'Senior Dev',      2),     -- reports to Bob
    (5, 'Eve',     'Junior Dev',      2),     -- reports to Bob
    (6, 'Frank',   'Sales Rep',       3);     -- reports to Charlie
```

```
Hierarchy:
Alice (CEO)
├── Bob (VP Engineering)
│   ├── Dave (Senior Dev)
│   └── Eve (Junior Dev)
└── Charlie (VP Sales)
    └── Frank (Sales Rep)
```

### Querying the Hierarchy

```sql
-- Find all direct reports of Bob (ManagerId = 2)
SELECT * FROM Employees WHERE ManagerId = 2;
-- Returns: Dave, Eve

-- Find an employee's manager
SELECT mgr.Name AS ManagerName, emp.Name AS EmployeeName
FROM Employees emp
JOIN Employees mgr ON emp.ManagerId = mgr.Id
WHERE emp.Name = 'Dave';
-- Returns: Bob is Dave's manager

-- Find the ENTIRE hierarchy (all levels) — requires a recursive CTE
WITH EmployeeHierarchy AS (
    -- Anchor: start with the root (CEO)
    SELECT Id, Name, Title, ManagerId, 0 AS Level
    FROM Employees
    WHERE ManagerId IS NULL

    UNION ALL

    -- Recursive: join children to their parents
    SELECT e.Id, e.Name, e.Title, e.ManagerId, eh.Level + 1
    FROM Employees e
    INNER JOIN EmployeeHierarchy eh ON e.ManagerId = eh.Id
)
SELECT Level, Name, Title
FROM EmployeeHierarchy
ORDER BY Level, Name;
```

```ad-warning
Self-referencing tables can create **circular references** — Employee A reports to Employee B, who reports to Employee A. The database's foreign key constraint does not prevent this. You must enforce acyclicity in application logic or with a CHECK constraint that prevents a row from referencing itself (`CHECK (ManagerId != Id)`) — though this only prevents the simplest case. Deep circular chains require application-level validation.
```

---

## Polymorphic Associations

- A **polymorphic association** occurs when a child table needs to relate to multiple different parent tables. For example, a `Comments` table where comments can be attached to `BlogPosts`, `Photos`, or `Videos` — three different entity types.

### Option A: Separate FK Columns (Nullable)

```sql
CREATE TABLE Comments (
    Id         INT PRIMARY KEY IDENTITY,
    Content    VARCHAR(1000) NOT NULL,
    BlogPostId INT NULL REFERENCES BlogPosts(Id),
    PhotoId    INT NULL REFERENCES Photos(Id),
    VideoId    INT NULL REFERENCES Videos(Id),
    -- Only one FK should be non-null per row
    CONSTRAINT CK_Comments_OneParent CHECK (
        (CASE WHEN BlogPostId IS NOT NULL THEN 1 ELSE 0 END +
         CASE WHEN PhotoId IS NOT NULL THEN 1 ELSE 0 END +
         CASE WHEN VideoId IS NOT NULL THEN 1 ELSE 0 END) = 1
    )
);
```

| Pros | Cons |
| --- | --- |
| Real foreign keys — referential integrity enforced | Adding a new parent type requires `ALTER TABLE` (new column) |
| Standard SQL — works everywhere | Many NULL columns; sparse; feels inelegant |
| Easy to query with JOINs | CHECK constraint complexity grows with each new type |

### Option B: Type + TypeId Pattern (Discriminator)

```sql
CREATE TABLE Comments (
    Id         INT PRIMARY KEY IDENTITY,
    Content    VARCHAR(1000) NOT NULL,
    ParentType VARCHAR(20) NOT NULL,  -- 'BlogPost', 'Photo', 'Video'
    ParentId   INT NOT NULL           -- the Id value in the parent table
);
```

| Pros | Cons |
| --- | --- |
| Clean, two-column design | ==No foreign key constraint possible== — `ParentId` can point to a non-existent row |
| Adding a new parent type is just a new `ParentType` value | Queries require dynamic logic based on `ParentType` |
| Used by many ORMs (e.g., EF Core) | Database cannot enforce referential integrity |

```ad-warning
Option B sacrifices referential integrity — the database has no way to verify that `ParentId = 42` with `ParentType = 'BlogPost'` actually corresponds to a row in the `BlogPosts` table. Orphaned records can and will occur. If you use this pattern, you must rely on application code to maintain integrity, which is inherently less reliable than database constraints.
```

### Option C: Separate Junction Tables

```sql
CREATE TABLE BlogPostComments (
    CommentId  INT PRIMARY KEY REFERENCES Comments(Id),
    BlogPostId INT NOT NULL REFERENCES BlogPosts(Id)
);

CREATE TABLE PhotoComments (
    CommentId INT PRIMARY KEY REFERENCES Comments(Id),
    PhotoId   INT NOT NULL REFERENCES Photos(Id)
);

CREATE TABLE VideoComments (
    CommentId INT PRIMARY KEY REFERENCES Comments(Id),
    VideoId   INT NOT NULL REFERENCES Videos(Id)
);
```

| Pros | Cons |
| --- | --- |
| Full referential integrity | More tables to manage |
| Clean foreign keys everywhere | Querying "all comments for any parent" requires UNION |
| Each relationship is explicit | Adding a new parent type requires a new junction table |

```ad-note
There is no perfect solution for polymorphic associations — each option involves a trade-off. **Option A** (separate FK columns) is the safest and most common for a small, fixed set of parent types. **Option C** (junction tables) is the most normalized. **Option B** (discriminator pattern) is the most flexible but sacrifices database-level integrity. Choose based on how critical referential integrity is versus how often new parent types are added.
```

---

## Anti-Patterns to Avoid

### 1. The God Table

- A **God table** is a single massive table with dozens or hundreds of columns that tries to store everything about every entity in one place.

```sql
-- BAD: God table — do NOT do this
CREATE TABLE Everything (
    Id                INT PRIMARY KEY,
    Type              VARCHAR(50),  -- 'User', 'Product', 'Order', ...
    Name              VARCHAR(100),
    Email             VARCHAR(200),
    Price             DECIMAL(10,2),
    Quantity          INT,
    OrderDate         DATETIME2,
    ShippingAddress   VARCHAR(500),
    ProductCategory   VARCHAR(100),
    UserRole          VARCHAR(50),
    -- ... 90 more columns, most NULL for any given row
);
```

- **Problems:** Most columns are NULL for any given row (a "User" row has no Price or Quantity). No meaningful constraints possible. Queries are confusing. Performance is terrible — every query reads a massive, sparse row.
- **Fix:** One table per entity, properly normalized. See [[01 - Normalization]].

### 2. Entity-Attribute-Value (EAV)

- The **EAV pattern** stores column names as data — each "attribute" is a row instead of a column.

```sql
-- BAD: EAV pattern
CREATE TABLE EntityAttributes (
    EntityId   INT,
    Attribute  VARCHAR(100),  -- 'Name', 'Price', 'Color', etc.
    Value      VARCHAR(500)   -- everything stored as a string
);
```

| EntityId | Attribute | Value |
| --- | --- | --- |
| 1 | Name | Widget |
| 1 | Price | 9.99 |
| 1 | Color | Blue |
| 2 | Name | Gadget |
| 2 | Price | 19.99 |

- **Problems:**
  - All values are strings — no type safety (Price "9.99" is a string, not a decimal)
  - No constraints possible — cannot enforce "Price must be positive"
  - Simple queries become complex pivots: getting Name and Price of entity 1 requires joining the table to itself
  - Performance is abysmal for any non-trivial query
  - Cannot use normal indexes effectively

```sql
-- Want to query: SELECT Name, Price FROM Products WHERE Price > 10
-- With EAV, you need:
SELECT
    n.Value AS Name,
    CAST(p.Value AS DECIMAL(10,2)) AS Price
FROM EntityAttributes n
JOIN EntityAttributes p ON n.EntityId = p.EntityId AND p.Attribute = 'Price'
WHERE n.Attribute = 'Name'
  AND CAST(p.Value AS DECIMAL(10,2)) > 10;
-- Unreadable, slow, and fragile
```

- **Fix:** Use proper tables with typed columns. If you need truly dynamic attributes (e.g., a configurable product catalog where different products have different properties), use a JSON column (supported in SQL Server 2016+, MariaDB 10.2+, PostgreSQL 9.2+) instead of EAV.

```ad-note
EAV is sometimes called "the inner-platform effect" — you are building a database system inside your database system. There are rare legitimate use cases (medical records with thousands of possible lab values, highly configurable SaaS platforms), but for the vast majority of applications, EAV is a mistake that will cause years of pain. If someone suggests it, push back hard and explore alternatives first.
```

### 3. Comma-Separated Values in a Column

- Storing multiple values in a single column as a comma-separated string — a direct violation of [[01 - Normalization#First Normal Form (1NF)|First Normal Form]].

```sql
-- BAD: CSV in a column
CREATE TABLE Users (
    Id    INT PRIMARY KEY,
    Name  VARCHAR(100),
    Roles VARCHAR(500)  -- 'Admin,Editor,Viewer'
);
```

- **Problems:** Cannot query ("find all Admins"), cannot index, cannot enforce valid values, cannot count, cannot join. Every operation requires string parsing.
- **Fix:** Junction/bridge table.

```sql
-- GOOD: Proper many-to-many with a junction table
CREATE TABLE Users (
    Id   INT PRIMARY KEY,
    Name VARCHAR(100)
);

CREATE TABLE Roles (
    Id   INT PRIMARY KEY,
    Name VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE UserRoles (
    UserId INT REFERENCES Users(Id),
    RoleId INT REFERENCES Roles(Id),
    PRIMARY KEY (UserId, RoleId)
);
```

### 4. No Foreign Keys — "We Enforce It in the App"

- Some teams omit foreign keys "for performance" or because "the application handles it."

```sql
-- BAD: No FK — CustomerId can point to a non-existent customer
CREATE TABLE Orders (
    Id         INT PRIMARY KEY IDENTITY,
    CustomerId INT NOT NULL  -- no FOREIGN KEY constraint!
);
```

- **Problems:**
  - Application bugs, direct SQL access, or data migrations can create orphan rows — orders pointing to customers that don't exist.
  - Data integrity degrades silently over time. You won't notice orphan records until a query produces wrong results or a JOIN returns no matches for rows that should have matches.
  - The "performance cost" of foreign keys is negligible compared to the cost of corrupted data. The FK check on insert is a single index lookup.

```ad-important
Foreign keys are not optional. The performance overhead is minimal — an indexed FK check is essentially free. The cost of *not* having foreign keys — orphan records, data corruption, application bugs that silently poison your data — is enormous and often irreversible. If someone argues against foreign keys for performance, they need to produce benchmarks showing a real, measured problem. In nearly all cases, the overhead is unmeasurable.
```

---

## Schema Design Checklist

When designing a new table or reviewing an existing schema, run through this checklist:

1. **Naming**: Do table and column names follow the project convention consistently?
2. **Primary key**: Does every table have a primary key? Is it a surrogate key (`Id INT IDENTITY`)?
3. **Foreign keys**: Does every relationship have a FK constraint? Are FK columns indexed?
4. **NOT NULL**: Is every column that should require a value marked `NOT NULL`?
5. **Defaults**: Do columns with sensible default values have DEFAULT constraints?
6. **Constraints**: Are business rules (positive prices, valid date ranges, allowed values) enforced with CHECK or FK to lookup tables?
7. **Normalization**: Is the schema in 3NF? Is each fact stored exactly once?
8. **Audit columns**: Does the table have `CreatedAt`/`CreatedBy`/`UpdatedAt`/`UpdatedBy`?
9. **Soft delete**: Does the table need soft delete? If so, is `IsDeleted` indexed?
10. **Indexes**: Are the most frequently queried columns indexed?

---

**Next:** See [[Performance and Administration]] folder.
