---
tags:
 - database
 - sql
 - reference
---

## SQL Vault Cross-Reference

The `D:\PROGRAMMING\SQL` vault contains detailed SQL syntax notes organized by topic. This note maps Database folder concepts to their corresponding SQL syntax notes to avoid duplication.

```ad-note
The **Database** folder covers concepts (what is a database, how design works, why things are done a certain way). The **SQL** folder covers syntax (how to write the actual queries). Use both together.
```


---

## Core Concepts → SQL Fundamentals

| Database note | SQL vault note |
|---|---|
| [[03 - Data Types]] | `SQL/1 - Fundamentals/Core/Data Types` |
| [[06 - NULL and Three-Valued Logic]] | `SQL/2 - Data Manipulation/Operators/IS NULL` |


---

## SQL Essentials → SQL Data Manipulation

| Database note | SQL vault note |
|---|---|
| [[01 - SELECT and FROM]] | `SQL/2 - Data Manipulation/Reading Data/SELECT` |
| [[02 - WHERE and Filtering]] | `SQL/2 - Data Manipulation/Reading Data/WHERE` + `SQL/2 - Data Manipulation/Operators/IN, BETWEEN, LIKE` |
| [[03 - INSERT, UPDATE, DELETE]] | `SQL/2 - Data Manipulation/Modifying Data/INSERT` + `UPDATE` + `DELETE` |
| [[04 - JOIN]] | `SQL/3 - Joins/Core/INNER JOIN` + `LEFT and RIGHT JOIN` + `Self Join` |
| [[05 - GROUP BY and Aggregation]] | `SQL/2 - Data Manipulation/Aggregation/GROUP BY` + `Aggregate Functions` + `HAVING` |
| [[06 - Subqueries and Common Table Expressions]] | `SQL/4 - Advanced Queries/Subqueries/Subqueries` + `Correlated Subqueries` + `Common Table Expressions` |
| [[07 - Views and Stored Procedures]] | `SQL/9 - Programmability/Views` + `Stored Procedures` |


---

## Design → SQL Database Design

| Database note | SQL vault note |
|---|---|
| [[01 - Normalization]] | `SQL/5 - Database Design/Normalization/Normal Forms` + `When to Denormalize` |
| [[02 - Indexes]] | `SQL/6 - Performance/Indexes/What are Indexes` + `Clustered vs Non-Clustered Indexes` + `When to Use Indexes` |
| [[03 - Constraints]] | `SQL/5 - Database Design/Constraints/Primary Key` + `Foreign Key` + `Unique, Not Null, Check, Default` |
| [[04 - Schema Design Patterns]] | `SQL/5 - Database Design/Table Management/CREATE TABLE` + `ALTER TABLE` + `DROP and TRUNCATE` |


---

## Performance and Administration → SQL Performance & Transactions

| Database note | SQL vault note |
|---|---|
| [[01 - Query Optimization]] | `SQL/6 - Performance/Query Analysis/EXPLAIN and Query Plans` |
| [[02 - Transactions and Concurrency]] | `SQL/7 - Transactions/Core/ACID Properties` + `Isolation Levels` + `BEGIN, COMMIT, ROLLBACK` |


---

## Topics Only in the SQL Vault (Not Covered Here)

These SQL-specific topics have no corresponding Database concept note — go directly to the SQL vault:

- **Window Functions** — `SQL/4 - Advanced Queries/Window Functions/` (ROW_NUMBER, RANK, LAG, LEAD)
- **Set Operations** — `SQL/4 - Advanced Queries/Set Operations/` (UNION, INTERSECT, EXCEPT)
- **Built-in Functions** — `SQL/8 - Built-in Functions/` (String, Date, Numeric, CAST/CONVERT, COALESCE)
- **ORDER BY / LIMIT / OFFSET** — `SQL/2 - Data Manipulation/Reading Data/`


---

## Topics Only in the Database Vault (Not Covered in SQL)

These concept notes have no corresponding SQL syntax note:

- [[01 - What Is a Database]] — foundational concept
- [[02 - Tables, Rows, and Columns]] — foundational concept
- [[04 - Primary Keys and Unique Identifiers]] — concept level (SQL vault has syntax-level Primary Key note)
- [[05 - Foreign Keys and Relationships]] — concept level with relationship types
- [[04 - Schema Design Patterns]] — practical patterns (soft delete, audit columns, anti-patterns)
- [[03 - Backup and Recovery]] — administration
- [[04 - Security Basics]] — administration
