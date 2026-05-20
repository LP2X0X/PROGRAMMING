---
tags: [sql, fundamentals, term]
---

- A **relational database** stores data in **tables** (also called relations). Each table represents an entity (e.g., customers, orders, products).
- A table is made up of **rows** (records/tuples) and **columns** (fields/attributes). Each row is a single entry, and each column defines a specific piece of data about that entry.
- A **schema** is the overall structure of the database — it defines what tables exist, what columns they have, what types those columns use, and how tables relate to each other.
- Tables are related to each other through **keys**:
  - A [[Primary Key]] uniquely identifies each row in a table.
  - A [[Foreign Key]] in one table references the primary key of another table, creating a relationship.
- Relationships between tables come in three forms:
  - **One-to-One**: one row in Table A relates to exactly one row in Table B (e.g., user → user_profile).
  - **One-to-Many**: one row in Table A relates to many rows in Table B (e.g., customer → orders). This is the most common relationship.
  - **Many-to-Many**: rows in Table A relate to many rows in Table B and vice versa. Implemented using a **junction table** (e.g., students ↔ courses via an enrollments table).

---

### Common RDBMS

| RDBMS        | Notes                                                        |
| ------------ | ------------------------------------------------------------ |
| **MySQL**    | Most popular open-source RDBMS. Default engine is InnoDB.    |
| **MariaDB**  | Fork of MySQL, fully compatible. Community-driven.           |
| **PostgreSQL** | Advanced open-source RDBMS. Strong standards compliance, rich features (JSON, arrays, window functions). |
| **SQL Server** | Microsoft's RDBMS. Uses T-SQL dialect. Integrated with .NET ecosystem. |
| **SQLite**   | Serverless, file-based. Great for embedded apps and prototyping. |
| **Oracle**   | Enterprise-grade. Uses PL/SQL dialect.                       |

```ad-tip
SQL syntax is mostly the same across databases, but each has dialect-specific features and function names. This vault notes dialect differences where they matter.
```
