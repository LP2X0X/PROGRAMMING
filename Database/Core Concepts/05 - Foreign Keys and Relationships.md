---
tags: [database, fundamentals, keys]
---

**Prerequisite:** [[04 - Primary Keys and Unique Identifiers]]

- Data in a relational database is split across multiple tables — users in one table, orders in another, products in a third. The question is: how do you **connect** them?
- The answer is the **foreign key** — a column in one table that references the primary key of another table. Foreign keys are the mechanism that makes a relational database *relational*.

---

### What Is a Foreign Key?

- A **foreign key (FK)** is a column (or set of columns) in a **child table** that references the **primary key** of a **parent table**:

```
Users (parent)                 Orders (child)
┌────┬──────────┐              ┌─────────┬────────┬────────┐
│ Id │ Name     │              │ OrderId │ UserId │ Total  │
├────┼──────────┤              ├─────────┼────────┼────────┤
│ 1  │ Long     │◄─────────── │ 101     │ 1      │ 59.99  │
│ 2  │ Pham     │◄─────────── │ 102     │ 2      │ 29.99  │
│ 3  │ Alice    │              │ 103     │ 1      │ 14.99  │
└────┴──────────┘              └─────────┴────────┴────────┘
                                UserId is a FOREIGN KEY
                                referencing Users.Id
```

- `Orders.UserId` is the foreign key. It "points to" `Users.Id` (the primary key of the Users table).
- Each order *belongs to* a user. The FK stores the user's ID, creating a link between the two tables.

```sql
CREATE TABLE Orders (
    OrderId INT PRIMARY KEY IDENTITY,
    UserId  INT NOT NULL,
    Total   DECIMAL(10, 2) NOT NULL,
    CONSTRAINT FK_Orders_Users FOREIGN KEY (UserId) REFERENCES Users(Id)
);
```

- The `CONSTRAINT FK_Orders_Users` part gives the foreign key a name. Naming constraints is a best practice — it makes error messages and schema management much clearer.

---

### What the Foreign Key Enforces — Referential Integrity

- The FK constraint enforces **referential integrity** — the guarantee that every reference (FK value) points to an existing record in the parent table:

| Operation | What the FK prevents |
| --- | --- |
| **INSERT into child** | Cannot insert an Order with `UserId = 999` if no User with `Id = 999` exists |
| **DELETE from parent** | Cannot delete User with `Id = 1` if Orders reference that user (by default) |
| **UPDATE parent PK** | Cannot change User's `Id` from 1 to 100 if Orders reference `Id = 1` (by default) |

```ad-important
title: Why Referential Integrity Matters
Without foreign keys, you get **orphaned records** — orders that point to users that no longer exist, enrollments for courses that were deleted, payments for orders that were removed. Orphaned records cause:

- Application errors (null reference exceptions when loading related data)
- Incorrect reports (aggregations include invalid references)
- Data that cannot be interpreted (what does `UserId = 47` mean if user 47 doesn't exist?)

Foreign keys prevent this entire class of bugs at the database level — no application code required.
```

---

### Three Types of Relationships

- Tables can be related in three ways. The relationship type determines where and how the foreign key is placed.

#### One-to-Many (1:N) — The Most Common

- **One** parent row relates to **many** child rows. This is the most common relationship in any database.
- The FK is placed in the **child** (the "many" side).

```
One User → Many Orders
One Department → Many Employees
One Category → Many Products
One Author → Many Books
```

```sql
-- One Department has many Employees
CREATE TABLE Departments (
    Id   INT PRIMARY KEY IDENTITY,
    Name VARCHAR(100) NOT NULL
);

CREATE TABLE Employees (
    Id           INT PRIMARY KEY IDENTITY,
    Name         VARCHAR(100) NOT NULL,
    DepartmentId INT NOT NULL,
    FOREIGN KEY (DepartmentId) REFERENCES Departments(Id)
);
```

- Employees "belong to" a Department. Each Department can have many Employees. Each Employee belongs to exactly one Department.

---

#### One-to-One (1:1)

- **One** row in Table A relates to **exactly one** row in Table B. Less common, but used for:
  - Splitting a wide table into two (e.g., separating rarely-accessed columns)
  - Optional extensions (e.g., a User may or may not have a Profile)
  - Security isolation (e.g., sensitive data in a separate table with different permissions)

```sql
-- One User has one Profile (optional)
CREATE TABLE Users (
    Id   INT PRIMARY KEY IDENTITY,
    Name VARCHAR(100) NOT NULL
);

CREATE TABLE Profiles (
    Id     INT PRIMARY KEY,  -- same ID as Users — shared PK
    Bio    NVARCHAR(MAX),
    Avatar VARCHAR(500),
    FOREIGN KEY (Id) REFERENCES Users(Id)  -- FK is also the PK
);
```

- Two common patterns for 1:1:
  1. **Shared primary key** — the child table's PK is also the FK (as above). Guarantees at most one profile per user.
  2. **FK with UNIQUE constraint** — a separate FK column with a UNIQUE constraint:

```sql
CREATE TABLE Profiles (
    ProfileId INT PRIMARY KEY IDENTITY,
    UserId    INT NOT NULL UNIQUE,  -- UNIQUE ensures 1:1
    Bio       NVARCHAR(MAX),
    FOREIGN KEY (UserId) REFERENCES Users(Id)
);
```

```ad-note
A 1:1 relationship is essentially a 1:N relationship with a `UNIQUE` constraint on the FK column, limiting the "many" side to exactly one.
```

---

#### Many-to-Many (M:N)

- **Many** rows in Table A relate to **many** rows in Table B. Examples:
  - Students ↔ Courses (a student takes many courses; a course has many students)
  - Products ↔ Tags (a product has many tags; a tag applies to many products)
  - Actors ↔ Movies (an actor appears in many movies; a movie has many actors)

- You **cannot** represent M:N directly with a single FK. You need a **junction table** (also called a bridge table, associative table, or linking table):

```
Students                Enrollments (junction)        Courses
┌────┬─────────┐       ┌───────────┬──────────┐      ┌────┬─────────────┐
│ Id │ Name    │       │ StudentId │ CourseId  │      │ Id │ CourseName  │
├────┼─────────┤       ├───────────┼──────────┤      ├────┼─────────────┤
│ 1  │ Long    │◄───── │ 1         │ 101      │ ────►│101 │ SQL 101     │
│ 2  │ Pham    │◄───── │ 1         │ 102      │ ────►│102 │ C# Basics   │
│ 3  │ Alice   │◄───── │ 2         │ 101      │      │103 │ Data Design │
└────┴─────────┘       │ 3         │ 103      │      └────┴─────────────┘
                       └───────────┴──────────┘
                       Two FKs: StudentId → Students.Id
                                CourseId → Courses.Id
```

```sql
CREATE TABLE Students (
    Id   INT PRIMARY KEY IDENTITY,
    Name VARCHAR(100) NOT NULL
);

CREATE TABLE Courses (
    Id         INT PRIMARY KEY IDENTITY,
    CourseName VARCHAR(200) NOT NULL
);

CREATE TABLE Enrollments (
    StudentId  INT NOT NULL,
    CourseId   INT NOT NULL,
    EnrolledAt DATETIME2 DEFAULT GETDATE(),
    Grade      CHAR(2) NULL,
    PRIMARY KEY (StudentId, CourseId),
    FOREIGN KEY (StudentId) REFERENCES Students(Id),
    FOREIGN KEY (CourseId) REFERENCES Courses(Id)
);
```

- The junction table has two foreign keys — one to each parent table.
- The composite primary key `(StudentId, CourseId)` ensures a student cannot be enrolled in the same course twice.
- Junction tables often carry **additional data** about the relationship — in this case, `EnrolledAt` and `Grade`.

```ad-tip
title: Naming Junction Tables
Common naming conventions for junction tables:

- **Descriptive noun**: `Enrollments`, `Assignments`, `Memberships` — best when the relationship has a real-world name
- **TableA_TableB**: `Student_Courses`, `Product_Tags` — simple and explicit
- **TableATableB**: `StudentCourses`, `ProductTags` — PascalCase concatenation

Pick one convention and use it consistently.
```

---

### ON DELETE and ON UPDATE — Referential Actions

- When a parent row is deleted or its PK is updated, what should happen to the child rows that reference it? The FK constraint supports several **referential actions**:

| Action | On DELETE | On UPDATE |
| --- | --- | --- |
| `NO ACTION` / `RESTRICT` | **Block** the delete — error if child rows exist | **Block** the update — error if child rows exist |
| `CASCADE` | **Delete** all child rows that reference the parent | **Update** FK values in child rows to match new parent PK |
| `SET NULL` | **Set** FK column to NULL in child rows | **Set** FK column to NULL in child rows |
| `SET DEFAULT` | **Set** FK column to its DEFAULT value | **Set** FK column to its DEFAULT value |

```sql
-- CASCADE: deleting a User also deletes all their Orders
FOREIGN KEY (UserId) REFERENCES Users(Id) ON DELETE CASCADE

-- SET NULL: deleting a Department sets Employees.DepartmentId to NULL
FOREIGN KEY (DepartmentId) REFERENCES Departments(Id) ON DELETE SET NULL

-- NO ACTION (default): can't delete a User who has Orders
FOREIGN KEY (UserId) REFERENCES Users(Id) ON DELETE NO ACTION

-- Mixed: cascade on delete, restrict on update
FOREIGN KEY (UserId) REFERENCES Users(Id) ON DELETE CASCADE ON UPDATE NO ACTION
```

```ad-warning
title: Use CASCADE with Extreme Caution
`ON DELETE CASCADE` is powerful but dangerous. Deleting a single parent row can silently delete thousands of child rows, grandchild rows, and so on through a chain of cascades.

Example of a cascade disaster:
1. Delete a `Department`
2. CASCADE deletes all `Employees` in that department
3. CASCADE deletes all `EmployeeAddresses` for those employees
4. CASCADE deletes all `Timesheets` for those employees
5. CASCADE deletes all `PayrollRecords` for those timesheets

One `DELETE` statement just wiped out payroll history.

**Best practice**: Use `NO ACTION` (the default) for most relationships. Use `CASCADE` only when the child rows genuinely have no meaning without the parent (e.g., order line items when an order is deleted). For important data, prefer **soft deletes** (setting an `IsDeleted` flag) over physical deletes.
```

```ad-note
title: NO ACTION vs RESTRICT
In most database systems, `NO ACTION` and `RESTRICT` behave identically — both block the operation if child rows exist. The technical difference:

- `RESTRICT` checks immediately
- `NO ACTION` checks at the end of the statement (allowing triggers to clean up first in some DBMS)

In SQL Server, only `NO ACTION` is supported (it is the default). In PostgreSQL and MySQL, both are available.
```

---

### Self-Referencing Foreign Keys

- A table can have a foreign key that references **its own primary key**. This is called a **self-referencing** or **recursive** relationship.
- Common use case: hierarchical data like organizational charts, category trees, or comment threads.

```sql
CREATE TABLE Employees (
    Id        INT PRIMARY KEY IDENTITY,
    Name      VARCHAR(100) NOT NULL,
    ManagerId INT NULL,  -- NULL for the CEO / top-level employee
    FOREIGN KEY (ManagerId) REFERENCES Employees(Id)
);
```

```
┌────┬──────────┬───────────┐
│ Id │ Name     │ ManagerId │
├────┼──────────┼───────────┤
│ 1  │ CEO      │ NULL      │  ← top of hierarchy
│ 2  │ VP Sales │ 1         │  ← reports to CEO
│ 3  │ VP Eng   │ 1         │  ← reports to CEO
│ 4  │ Dev Lead │ 3         │  ← reports to VP Eng
│ 5  │ Dev      │ 4         │  ← reports to Dev Lead
└────┴──────────┴───────────┘
```

- The FK `ManagerId` references `Employees.Id` — the same table. The top-level employee has `ManagerId = NULL` (no manager).

---

### Foreign Key and Indexes

- The DBMS does **not** automatically create an index on FK columns in all systems:
  - **SQL Server**: does NOT auto-create an index on FK columns
  - **MySQL / InnoDB**: DOES auto-create an index on FK columns

```ad-important
title: Always Index Your Foreign Key Columns
If your DBMS does not auto-index FK columns (like SQL Server), create the index manually:

- Without an index on the FK column, the DBMS must do a **full table scan** of the child table every time you:
  - Join the tables
  - Delete or update a parent row (to check for child references)
- This can make joins and cascaded operations extremely slow on large tables.

Always create a non-clustered index on every FK column.
```

```sql
-- SQL Server: manually index the FK column
CREATE NONCLUSTERED INDEX IX_Orders_UserId ON Orders(UserId);
```

---

### Practical Example — Complete Schema

- Here is a small but realistic schema showing all three relationship types:

```sql
-- Users table (parent)
CREATE TABLE Users (
    Id    INT PRIMARY KEY IDENTITY,
    Name  NVARCHAR(100) NOT NULL,
    Email VARCHAR(200) NOT NULL UNIQUE
);

-- Profiles table (1:1 with Users)
CREATE TABLE Profiles (
    UserId INT PRIMARY KEY,
    Bio    NVARCHAR(MAX),
    FOREIGN KEY (UserId) REFERENCES Users(Id) ON DELETE CASCADE
);

-- Orders table (1:N — one User has many Orders)
CREATE TABLE Orders (
    Id     INT PRIMARY KEY IDENTITY,
    UserId INT NOT NULL,
    Total  DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (UserId) REFERENCES Users(Id) ON DELETE NO ACTION
);

-- Products table
CREATE TABLE Products (
    Id    INT PRIMARY KEY IDENTITY,
    Name  NVARCHAR(200) NOT NULL,
    Price DECIMAL(10, 2) NOT NULL
);

-- OrderItems table (junction — M:N between Orders and Products)
CREATE TABLE OrderItems (
    OrderId   INT NOT NULL,
    ProductId INT NOT NULL,
    Quantity  INT NOT NULL DEFAULT 1,
    UnitPrice DECIMAL(10, 2) NOT NULL,
    PRIMARY KEY (OrderId, ProductId),
    FOREIGN KEY (OrderId) REFERENCES Orders(Id) ON DELETE CASCADE,
    FOREIGN KEY (ProductId) REFERENCES Products(Id) ON DELETE NO ACTION
);
```

- This schema demonstrates:
  - **1:1**: Users ↔ Profiles (shared PK pattern)
  - **1:N**: Users → Orders (FK in child table)
  - **M:N**: Orders ↔ Products (via OrderItems junction table)
  - Appropriate referential actions: CASCADE for tightly coupled data (profile dies with user, order items die with order), NO ACTION for loosely coupled data (can't delete a user with orders, can't delete a product that's been ordered)

---

### Common Mistakes

```ad-warning
title: Foreign Key Anti-Patterns to Avoid
1. **No foreign keys at all** — relying on application code for referential integrity. The application has bugs; the database doesn't.
2. **Circular cascades** — Table A cascades to B, B cascades to C, C cascades to A. The DBMS will reject this with a "cycle" error.
3. **FK to non-unique column** — a foreign key must reference a column with a PRIMARY KEY or UNIQUE constraint. You cannot FK to an arbitrary column.
4. **Mismatched data types** — the FK column and the referenced PK column must have the **exact same data type**. `INT` FK referencing `BIGINT` PK will fail or cause implicit conversion.
5. **Missing indexes on FK columns** — especially in SQL Server. See the indexing section above.
6. **Using CASCADE everywhere** — makes deletes dangerous and unpredictable. Use NO ACTION as the default.
```

---

**Next:** [[06 - NULL and Three-Valued Logic]]
