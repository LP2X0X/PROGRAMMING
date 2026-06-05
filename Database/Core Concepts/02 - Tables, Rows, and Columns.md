---
tags: [database, fundamentals, concepts]
---

**Prerequisite:** [[01 - What Is a Database]]

- A relational database stores all data in **tables** (also called **relations** in formal relational theory). Every piece of data lives in a table.
- A table is a two-dimensional structure made up of **columns** (defining *what* data is stored) and **rows** (containing the *actual* data).

---

### Anatomy of a Table

- A table looks like a spreadsheet, but with strict rules:

```
Users table:
┌────┬──────────┬─────┬─────────────────┐
│ Id │ Name     │ Age │ Email           │
├────┼──────────┼─────┼─────────────────┤
│ 1  │ Long     │ 28  │ long@email.com  │
│ 2  │ Pham     │ 25  │ pham@email.com  │
│ 3  │ Alice    │ 30  │ alice@email.com │
└────┴──────────┴─────┴─────────────────┘
```

- **Table name**: `Users` — describes the entity this table represents. Table names should be descriptive nouns.
- **Columns** (`Id`, `Name`, `Age`, `Email`): define the *attributes* of the entity. Each column has a name, a [[03 - Data Types|data type]], and optional constraints.
- **Rows**: each row is one *instance* of the entity — one user, one order, one product. The table above has 3 rows.
- **Values**: the data at a specific row-column intersection. Row 1, column `Name` = `Long`.

---

### Terminology Translation

- Database terminology can be confusing because there are multiple naming conventions from different eras and contexts:

| Spreadsheet term | Database term (common) | Relational theory term | Meaning |
| --- | --- | --- | --- |
| Sheet | Table | Relation | A collection of related data |
| Column header | Column / Field | Attribute | Defines what kind of data is stored |
| Row | Row / Record | Tuple | A single entry in the table |
| Cell | Value | Attribute value | A single piece of data at a row/column intersection |
| Spreadsheet file | Database | Database / Schema | The container for all tables |

```ad-note
In practice, "column" and "field" are used interchangeably. "Row" and "record" are used interchangeably. The formal relational theory terms (relation, tuple, attribute) appear in academic textbooks and certification exams but are rarely used in everyday development.
```

---

### Schema — The Blueprint of a Table

- The **schema** is the *structure* of a table — its column names, data types, and constraints. It is defined once when you create the table, and the DBMS enforces it on every row.
- Think of the schema as a blueprint or contract: "this table will have these columns, with these types, and these rules."

```sql
CREATE TABLE Users (
    Id       INT          NOT NULL,
    Name     VARCHAR(100) NOT NULL,
    Age      INT          NULL,
    Email    VARCHAR(200) NOT NULL
);
```

- This schema says:
  - `Id` is an integer, required (cannot be NULL)
  - `Name` is a string up to 100 characters, required
  - `Age` is an integer, optional (can be NULL — meaning "unknown")
  - `Email` is a string up to 200 characters, required

```ad-important
Unlike a spreadsheet where you can put any value in any cell, a database **enforces the schema**. If you try to insert text into an `INT` column, the DBMS will reject the operation with an error. This enforcement is one of the most important advantages of a database over files.
```

---

### Column Data Types

- Every column has a **data type** that determines what kind of values it can hold. The DBMS enforces this — you cannot put text in an integer column, or a date in a boolean column.
- Data types fall into several categories: numeric, string, date/time, binary, and boolean. The exact type names vary by database system.
- Choosing the right data type matters for:
  - **Storage efficiency** — `SMALLINT` (2 bytes) vs `BIGINT` (8 bytes)
  - **Query performance** — comparing integers is faster than comparing strings
  - **Data integrity** — the type prevents invalid data at the DBMS level
  - **Semantics** — a `DATE` column enables date arithmetic; a `VARCHAR` storing dates does not

```ad-tip
Data types are covered in detail in [[03 - Data Types]]. For now, understand that every column must have one, and it is enforced by the DBMS.
```

---

### Rows — Each Row Is Unique

- Each row represents a single, distinct record. No two rows in a table should be identical.
- This uniqueness is enforced by a **primary key** — a column (or combination of columns) whose value is unique for every row. See [[04 - Primary Keys and Unique Identifiers]].
- Without a primary key, you could end up with duplicate rows and no way to tell them apart:

```
-- BAD: no primary key, duplicate rows
┌──────────┬─────┬─────────────────┐
│ Name     │ Age │ Email           │
├──────────┼─────┼─────────────────┤
│ Long     │ 28  │ long@email.com  │
│ Long     │ 28  │ long@email.com  │  ← which Long is this?
└──────────┴─────┴─────────────────┘
```

```ad-warning
Always define a primary key on every table. A table without a primary key is called a **heap** in some database systems. Heaps have no guaranteed row order and cannot be efficiently referenced by other tables. There are almost no legitimate reasons to have a table without a primary key.
```

---

### Tables Are Independent (Until You Connect Them)

- Each table stores data about one entity (users, orders, products). Tables are **independent** by default — they don't know about each other.
- To connect tables — for example, to know which user placed which order — you use **keys**:
  - A **primary key** uniquely identifies rows within a table. See [[04 - Primary Keys and Unique Identifiers]].
  - A **foreign key** in one table references the primary key of another table, creating a relationship. See [[05 - Foreign Keys and Relationships]].

```
Users                          Orders
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

- This separation of data into multiple related tables is called **normalization** — it eliminates data redundancy and keeps data consistent.

---

### Table Naming Conventions

- There is no universal standard, but here are the most common conventions:

| Convention | Example | Used by |
| --- | --- | --- |
| **PascalCase singular** | `User`, `OrderItem` | Entity Framework (C#/.NET), many ORMs |
| **PascalCase plural** | `Users`, `OrderItems` | Ruby on Rails, some ORMs |
| **snake_case singular** | `user`, `order_item` | PostgreSQL community convention |
| **snake_case plural** | `users`, `order_items` | Many web frameworks |

```ad-tip
Pick one convention and stick with it across your entire database. Mixing conventions (e.g., `Users` and `order_item`) makes the schema confusing. If you are using an ORM, follow the convention your ORM expects.
```

- Column naming follows the same idea. Common conventions:
  - `PascalCase`: `FirstName`, `CreatedAt` (common in SQL Server / .NET)
  - `snake_case`: `first_name`, `created_at` (common in PostgreSQL / MySQL)
  - Prefix booleans: `IsActive`, `HasDiscount`, `is_active`, `has_discount`

---

### How the DBMS Stores Tables Internally

- You interact with tables as a logical two-dimensional grid, but the DBMS stores them differently on disk:
  - **Pages/blocks**: data is stored in fixed-size pages (typically 8 KB in SQL Server, 16 KB in MySQL/InnoDB). Each page holds multiple rows.
  - **Row storage**: most relational databases store data row by row (row-oriented storage). All columns of a single row are stored together on the same page.
  - **Extents**: groups of pages allocated together for performance (e.g., 8 pages = 1 extent in SQL Server).
- You do not need to manage pages or extents manually — the DBMS handles all of this. But understanding that data lives in pages helps explain why:
  - Smaller data types = more rows per page = faster scans
  - Indexes create separate page structures for fast lookups
  - Very wide rows (many columns or large columns) can hurt performance

```ad-note
Column-oriented databases (like ClickHouse, Amazon Redshift) store data column by column instead of row by row. This is much faster for analytical queries that scan a few columns across millions of rows, but slower for transactional queries that read/write full rows. Traditional RDBMS like SQL Server, MySQL, and PostgreSQL are row-oriented.
```

---

**Next:** [[03 - Data Types]]
