---
tags: [database, fundamentals, keys]
---

**Prerequisite:** [[03 - Data Types]]

- The fundamental problem: how do you identify a *specific* row in a table? If two users are both named "Long" and both are age 28, which one do you mean?
- The answer is a **primary key** — a column (or combination of columns) that **uniquely identifies each row**. No two rows can have the same primary key value, and it can never be NULL.

---

### What Is a Primary Key?

- A **primary key (PK)** is a constraint on one or more columns that enforces two rules:
  1. **Uniqueness** — every row must have a different PK value
  2. **Not NULL** — the PK value must always be present (no unknowns)

```sql
CREATE TABLE Users (
    Id    INT PRIMARY KEY,
    Name  VARCHAR(100) NOT NULL,
    Email VARCHAR(200) NOT NULL
);
```

- In this table, `Id` is the primary key. The DBMS guarantees:
  - You cannot insert two rows with the same `Id`
  - You cannot insert a row without an `Id` value

```ad-note
A table can have **only one primary key**. However, it can have multiple [[#UNIQUE Constraint|UNIQUE constraints]] — those enforce uniqueness on other columns but are not the "official" row identifier.
```

---

### Auto-Increment — Let the DBMS Generate the Key

- Manually assigning unique IDs to every row is impractical. Instead, let the DBMS generate them automatically using **auto-increment**:

```sql
-- SQL Server: IDENTITY(seed, increment)
CREATE TABLE Users (
    Id    INT IDENTITY(1, 1) PRIMARY KEY,  -- starts at 1, increments by 1
    Name  VARCHAR(100) NOT NULL,
    Email VARCHAR(200) NOT NULL
);

-- MySQL / MariaDB: AUTO_INCREMENT
CREATE TABLE Users (
    Id    INT AUTO_INCREMENT PRIMARY KEY,
    Name  VARCHAR(100) NOT NULL,
    Email VARCHAR(200) NOT NULL
);
```

- When you insert a row, you **omit** the PK column — the DBMS fills it in:

```sql
INSERT INTO Users (Name, Email) VALUES ('Long', 'long@email.com');
-- Id is automatically assigned as 1

INSERT INTO Users (Name, Email) VALUES ('Pham', 'pham@email.com');
-- Id is automatically assigned as 2
```

```ad-warning
title: Auto-Increment Gaps Are Normal
Auto-increment values can have gaps. If you insert a row and then delete it, the ID is not reused. If a transaction rolls back, the ID is consumed but the row doesn't exist. If concurrent inserts happen, IDs may not be sequential.

**This is by design.** The primary key's job is to be *unique*, not *sequential* or *gapless*. Never write application logic that assumes consecutive IDs. Never use IDs for counting rows — use `COUNT(*)` instead.
```

---

### Natural Key vs Surrogate Key

- There are two fundamentally different approaches to choosing a primary key:

| | Natural Key | Surrogate Key |
| --- | --- | --- |
| **Definition** | A column with real-world meaning that happens to be unique | A system-generated column with no business meaning |
| **Examples** | Email address, Social Security Number, ISBN, country code | Auto-incrementing integer, GUID/UUID |
| **Pros** | Meaningful — you can identify the row by looking at the key. No extra column needed. | Never changes. Guaranteed unique. No business logic dependency. Simple. |
| **Cons** | Can change (user changes email). May be composite (multiple columns). May not be truly unique (names). Privacy concerns (SSN). | Meaningless number — requires a lookup to understand the row. Extra column. |
| **Best practice** | Use as a `UNIQUE` constraint, not as the PK | Use as the PK (the standard approach in almost all modern databases) |

```ad-important
title: Industry Best Practice — Use Surrogate Keys
The overwhelming consensus in database design is: **use a surrogate key (auto-increment INT) as the primary key**, and enforce natural uniqueness with a separate `UNIQUE` constraint.

Example:
- PK: `Id INT IDENTITY`
- UNIQUE: `Email VARCHAR(200) UNIQUE`

This gives you the best of both worlds — a stable, simple PK that never changes, plus a business rule that emails must be unique.
```

#### Why Natural Keys Are Risky as Primary Keys

- **They change.** A user changes their email. An ISBN gets reassigned. A company changes its tax ID. When the PK changes, *every foreign key pointing to it* must also change — a cascading nightmare.
- **They're not always unique.** Names are not unique. Phone numbers get recycled. Even emails can change ownership.
- **They can be composite.** Using `(FirstName, LastName, BirthDate)` as a PK makes joins verbose and indexes large.
- **Privacy concerns.** Using SSN or national ID as a PK means that value appears in every related table and every join — a security and compliance risk.

---

### GUID / UUID as Primary Key

- A **GUID** (Globally Unique Identifier) or **UUID** (Universally Unique Identifier) is a 128-bit randomly generated value:

```
'6F9619FF-8B86-D011-B42D-00CF4FC964FF'
```

```sql
-- SQL Server
CREATE TABLE Users (
    Id UNIQUEIDENTIFIER DEFAULT NEWID() PRIMARY KEY,
    Name VARCHAR(100) NOT NULL
);

-- MySQL / MariaDB
CREATE TABLE Users (
    Id CHAR(36) DEFAULT (UUID()) PRIMARY KEY,
    Name VARCHAR(100) NOT NULL
);
```

| Pros | Cons |
| --- | --- |
| Globally unique — safe for distributed systems, merging databases | **Large**: 16 bytes vs 4 bytes for INT |
| Can be generated client-side (no round-trip to DB) | **Bad for clustered indexes**: random values cause page splits and fragmentation |
| Hard to guess — slight security benefit | **Hard to debug**: `WHERE Id = '6F9619FF-8B86-D011-B42D-00CF4FC964FF'` vs `WHERE Id = 42` |
| Merging data from multiple databases is trivial | **Slower joins**: comparing 16-byte values is slower than 4-byte integers |

```ad-tip
title: NEWSEQUENTIALID() in SQL Server
If you must use GUIDs as primary keys in SQL Server, use `NEWSEQUENTIALID()` instead of `NEWID()`. It generates GUIDs that are **sequential** — each new GUID is larger than the last. This dramatically reduces index fragmentation and page splits compared to random GUIDs.

However, `INT IDENTITY` is still faster and smaller in almost every scenario.
```

#### When to Use GUIDs

- **Distributed systems** — multiple database nodes generating IDs independently without coordination
- **Data merging** — combining data from multiple sources where auto-increment IDs would collide
- **Offline-first apps** — mobile or desktop apps that create records offline and sync later
- **Security-sensitive URLs** — using IDs in URLs where sequential integers would let users guess other records (`/users/1`, `/users/2`, ...)

For all other cases, prefer `INT IDENTITY` / `INT AUTO_INCREMENT`.

---

### Composite Primary Key

- A **composite primary key** spans multiple columns. The *combination* of values must be unique, though individual columns can repeat.

```sql
CREATE TABLE Enrollments (
    StudentId INT,
    CourseId  INT,
    EnrolledAt DATETIME2 DEFAULT GETDATE(),
    PRIMARY KEY (StudentId, CourseId)  -- the combination must be unique
);
```

- This means:
  - Student 1 in Course 101 → allowed
  - Student 1 in Course 102 → allowed (same student, different course)
  - Student 2 in Course 101 → allowed (different student, same course)
  - Student 1 in Course 101 again → **rejected** (duplicate combination)

```ad-note
Composite primary keys are most common in **junction tables** (also called bridge tables or associative tables) that implement many-to-many relationships. See [[05 - Foreign Keys and Relationships#Many-to-Many Relationships]].
```

#### Composite PK vs Surrogate PK + Unique Constraint

- Some designers prefer adding a surrogate PK even to junction tables:

```sql
-- Option A: Composite PK (traditional, purist)
CREATE TABLE Enrollments (
    StudentId INT,
    CourseId  INT,
    PRIMARY KEY (StudentId, CourseId)
);

-- Option B: Surrogate PK + unique constraint (pragmatic)
CREATE TABLE Enrollments (
    Id        INT IDENTITY PRIMARY KEY,
    StudentId INT,
    CourseId  INT,
    UNIQUE (StudentId, CourseId)
);
```

- Option A is simpler and wastes no space. Option B is easier to reference from other tables and works better with some ORMs that expect a single-column PK.

---

### UNIQUE Constraint

- A `UNIQUE` constraint enforces that all values in a column (or combination of columns) are distinct — like a primary key, but with key differences:

| | Primary Key | UNIQUE Constraint |
| --- | --- | --- |
| **NULLs** | Never allows NULL | Allows NULL (one NULL in SQL Server, multiple NULLs in MySQL/PostgreSQL) |
| **How many per table** | Exactly one | As many as needed |
| **Creates index** | Yes — clustered index by default (SQL Server) | Yes — non-clustered index by default |
| **Purpose** | The official row identifier | Enforce a business rule (e.g., email must be unique) |

```sql
CREATE TABLE Users (
    Id    INT IDENTITY PRIMARY KEY,          -- the official row ID
    Email VARCHAR(200) NOT NULL UNIQUE,      -- business rule: no duplicate emails
    Phone VARCHAR(20) UNIQUE                 -- optional but unique if present
);
```

```ad-warning
title: NULL Behavior in UNIQUE Columns Varies by DBMS
- **SQL Server**: allows only **one** NULL in a UNIQUE column (treats NULLs as equal for uniqueness)
- **PostgreSQL** and **MySQL**: allow **multiple** NULLs in a UNIQUE column (treats NULLs as distinct)

This is a subtle but important difference when designing cross-platform schemas. To be safe, combine `UNIQUE` with `NOT NULL` when the column should always have a value.
```

---

### Identity Column Internals

- Understanding how the DBMS manages auto-increment values helps avoid common pitfalls:

#### SQL Server IDENTITY

- `IDENTITY(seed, increment)` — `seed` is the starting value, `increment` is the step.
- The current identity value is cached in memory. If the server crashes, the cache can be lost, causing a gap in the next restart.
- To get the last generated identity value:

```sql
-- After an INSERT in the same session:
SELECT SCOPE_IDENTITY();     -- recommended: returns the last identity in the current scope
SELECT @@IDENTITY;            -- returns the last identity in the current session (including triggers)
SELECT IDENT_CURRENT('Users'); -- returns the last identity for a specific table (any session)
```

```ad-warning
title: SCOPE_IDENTITY() vs @@IDENTITY
Always use `SCOPE_IDENTITY()`, not `@@IDENTITY`. If your `INSERT` fires a trigger that inserts into another table with an IDENTITY column, `@@IDENTITY` returns the trigger's identity value, not yours. `SCOPE_IDENTITY()` returns only the identity generated in *your* scope.
```

#### MySQL / MariaDB AUTO_INCREMENT

- `AUTO_INCREMENT` starts at 1 by default and increments by 1.
- The current value is stored in memory (InnoDB) and reset on server restart to `MAX(id) + 1` (prior to MySQL 8.0). In MySQL 8.0+, it is persisted.
- To get the last generated value:

```sql
SELECT LAST_INSERT_ID();  -- returns the last auto-increment value for the current connection
```

---

### Practical Guidelines

```ad-tip
title: Summary of Best Practices
1. **Every table must have a primary key** — no exceptions in production databases
2. **Use surrogate keys** (`INT IDENTITY` / `INT AUTO_INCREMENT`) as the PK for almost all tables
3. **Enforce natural uniqueness with `UNIQUE` constraints** — don't rely on application code to check for duplicates
4. **Use `BIGINT` only when `INT` is genuinely too small** — `INT` supports ~2.1 billion rows, which is sufficient for most tables
5. **Avoid GUIDs as PKs unless you have a specific reason** (distributed systems, data merging, security)
6. **Use composite PKs only for junction tables** — and even then, consider a surrogate PK if ORMs demand it
7. **Use `SCOPE_IDENTITY()` (SQL Server) or `LAST_INSERT_ID()` (MySQL)** to retrieve the generated key after an insert
```

---

**Next:** [[05 - Foreign Keys and Relationships]]
