---
tags: [database, fundamentals, concepts]
---

- A **database** is a structured collection of data stored electronically. It is organized so that data can be efficiently stored, retrieved, updated, and deleted.
- A database is *not* just a file sitting on disk. What makes it a database is that it is managed by a specialized piece of software called a **Database Management System (DBMS)**.

---

### DBMS vs Database

- The **DBMS** is the *software* — it is the engine that creates, manages, and controls access to the database. Examples: SQL Server, MariaDB, PostgreSQL, MySQL, Oracle, SQLite.
- The **database** is the *data* — the actual tables, rows, relationships, and constraints that the DBMS manages.
- Think of it this way: the DBMS is the *librarian*, and the database is the *library's collection of books*. You don't interact with the books directly — you ask the librarian (DBMS) to find, add, or remove books for you.

```ad-note
In everyday conversation, people use "database" to mean both the software and the data interchangeably. Technically, when someone says "I installed PostgreSQL," they installed a DBMS. When they say "I created a database," they created a data container inside that DBMS. A single DBMS instance can host *many* databases.
```

---

### Why Not Just Use Files?

- Before databases, applications stored data in flat files — CSV, JSON, XML, Excel spreadsheets, or custom binary formats.
- Files work fine for small, single-user scenarios. But they break down as requirements grow. Here is why:

| Feature | Files (CSV, JSON, Excel) | Database (DBMS) |
| --- | --- | --- |
| **Concurrent access** | Corrupts data or locks the file — only one writer at a time | Handles thousands of simultaneous readers and writers safely |
| **Querying complex data** | Manual parsing, custom code for every query | SQL — a powerful, standardized query language |
| **Data integrity** | No enforcement — nothing stops invalid data | Constraints, foreign keys, transactions enforce correctness |
| **Performance at scale** | Degrades fast as file size grows (linear scans) | Indexes, query optimizer, buffer pools — designed for scale |
| **Security** | OS-level file permissions (all or nothing) | Row-level, column-level, user roles, fine-grained access control |
| **Relationships** | Manual linking across files, error-prone | Built-in relational model with foreign keys |
| **Backup & recovery** | Copy the file, hope nothing changed mid-copy | Point-in-time recovery, transaction logs, automated backups |
| **Data redundancy** | Easy to accidentally duplicate data across files | Normalization eliminates redundancy by design |

```ad-warning
A common beginner mistake is thinking "my app is small, I'll just use a JSON file." This works until you need concurrent access, data validation, or complex queries — at which point migrating to a database is far more painful than starting with one. If your data has relationships or needs to be queried in multiple ways, start with a database.
```

---

### Relational vs Non-Relational Databases

- There are two broad families of databases. This note series focuses on **relational databases**, but it is important to know both exist.

| | Relational (SQL) | Non-Relational (NoSQL) |
| --- | --- | --- |
| **Data model** | Tables with rows and columns | Documents, key-value pairs, graphs, wide-column stores |
| **Query language** | SQL (Structured Query Language) | Varies — MongoDB uses MQL, Redis uses commands, etc. |
| **Schema** | Fixed schema — columns and types defined upfront | Flexible / schemaless — each record can have different fields |
| **Relationships** | First-class support via foreign keys and joins | Typically embedded or denormalized — joins are expensive or unavailable |
| **Consistency** | Strong consistency (ACID transactions) | Often eventual consistency (BASE) for scalability |
| **Examples** | SQL Server, MariaDB, PostgreSQL, MySQL, SQLite, Oracle | MongoDB, Redis, Cassandra, DynamoDB, Neo4j |
| **Best for** | Structured data, complex queries, strong consistency, transactional systems | Unstructured/semi-structured data, massive horizontal scale, flexible schemas, real-time caching |

```ad-note
Relational databases have been the foundation of data management since the 1970s and remain the dominant choice for most applications. Understanding relational concepts is essential even if you later work with NoSQL — many NoSQL systems borrow relational ideas, and most business applications still run on relational databases.
```

---

### The Client-Server Model

- Most database systems follow a **client-server architecture**:

```
Your Application (client)
        │
        │  SQL commands sent over TCP/IP connection
        ▼
Database Server (DBMS process)
        │
        │  Reads/writes managed by the DBMS engine
        ▼
Data Files on Disk (the actual database)
```

- **Client**: your application, a command-line tool (like `mysql` or `sqlcmd`), or a GUI tool (like SQL Server Management Studio, DBeaver, HeidiSQL). The client sends SQL commands to the server.
- **Server**: the DBMS process running on a machine (local or remote). It receives SQL commands, parses them, optimizes them, executes them against the data files, and returns results.
- **Data files**: the physical files on disk where the DBMS stores tables, indexes, logs, and metadata. You never edit these directly — always go through the DBMS.

```ad-important
SQLite is the notable exception — it is an **embedded** database with no separate server process. The DBMS is a library linked directly into your application, and it reads/writes a single file on disk. This makes SQLite ideal for mobile apps, desktop apps, and prototyping, but unsuitable for high-concurrency server applications.
```

- The connection between client and server uses a **connection string** — a formatted string containing the server address, port, database name, and authentication credentials:

```
Server=localhost;Port=3306;Database=myapp;User=root;Password=secret;
```

---

### Key Terminology Summary

| Term | Meaning |
| --- | --- |
| **Database** | A structured collection of data, organized into tables |
| **DBMS** | The software that manages the database (SQL Server, MariaDB, etc.) |
| **SQL** | Structured Query Language — the standard language for querying relational databases |
| **Schema** | The structure of a database — what tables exist, their columns, types, and relationships |
| **Client** | The application or tool that sends SQL commands to the DBMS |
| **Server** | The DBMS process that executes SQL commands and manages data |
| **RDBMS** | Relational Database Management System — a DBMS that uses the relational model |

---

**Next:** [[02 - Tables, Rows, and Columns]]
