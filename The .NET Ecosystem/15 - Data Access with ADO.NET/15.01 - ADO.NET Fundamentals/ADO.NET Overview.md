---
tags:
  - csharp
  - ado-net
  - data-access
aliases:
  - ADO.NET Introduction
  - ADO.NET Architecture
---

## ADO.NET Overview

```ad-note
title: What You'll Learn
ADO.NET is the foundational data access technology in .NET. This note covers what it is, how it compares to Entity Framework, its two-layer architecture (connected and disconnected), and the key namespaces you'll work with. Understanding this architecture is essential before diving into individual classes like [[DbConnection]], [[DbCommand]], or [[DbDataReader]].
```

---

## Table of Contents

- [[#What is ADO.NET?]]
- [[#ADO.NET vs Entity Framework]]
- [[#The Two Layers of ADO.NET]]
  - [[#Connected Layer]]
  - [[#Disconnected Layer]]
- [[#Architecture Diagram]]
- [[#Key Namespaces]]
- [[#When to Use ADO.NET]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]

---

## What Is a Database Server?

Before diving into ADO.NET, you need to understand what you're connecting **to**. A database server is a software program (running as a service/process) that stores, manages, and retrieves data. Your C# application communicates with it over a network protocol.

**SQL Server** is Microsoft's relational database management system (RDBMS). It runs as a Windows service (or Linux daemon) that listens for connections on a port (default 1433). When your code calls `conn.Open()`, it establishes a TCP connection to this service.

| Database Server | Vendor | Default Port | Common Use |
|---|---|---|---|
| SQL Server | Microsoft | 1433 | Enterprise .NET apps, Azure |
| MySQL / MariaDB | Oracle / MariaDB Foundation | 3306 | Web apps, open source |
| PostgreSQL | Community | 5432 | Advanced features, GIS, open source |
| SQLite | Public domain | N/A (file-based) | Mobile, embedded, local apps |
| Oracle Database | Oracle | 1521 | Enterprise, banking |

```ad-note
SQLite is different — it's not a server. It's a library that reads/writes directly to a file. No network, no service, no port. Great for local storage, mobile apps, and prototyping.
```

The database server handles:
- Storing data on disk (tables, indexes)
- Executing SQL queries
- Managing concurrent access (locks, transactions)
- Authentication and authorization
- Replication and backups

Your C# code never touches the data files directly. It sends SQL commands to the server, and the server sends results back. ADO.NET is the .NET library that manages this communication.


---

## What is ADO.NET?

**ADO.NET** (ActiveX Data Objects for .NET) is the ==core data access technology== in the .NET Base Class Library (BCL). It provides a set of classes for communicating with relational databases and other data sources directly from C# code.

Key characteristics:

- **It is not an ORM** — you write SQL yourself and manually map results to objects
- **Part of the BCL** — the abstract base classes (`System.Data`, `System.Data.Common`) ship with the .NET runtime; concrete providers are NuGet packages
- **Provider-based architecture** — each database vendor supplies a **data provider** (a set of classes implementing the ADO.NET interfaces) so the same programming model works across SQL Server, MySQL, PostgreSQL, SQLite, Oracle, and more
- **Two programming models** — a connected model for streaming reads and a disconnected model for in-memory caching
- **Synchronous and asynchronous** — all I/O operations have `async` counterparts (`OpenAsync`, `ExecuteReaderAsync`, etc.)

```ad-info
title: Historical Context
ADO.NET shipped with .NET Framework 1.0 in 2002. The name "ADO" is inherited from the older COM-based ADO technology, but the API is completely different. In modern .NET (5+), the abstract base classes remain in `System.Data.Common`, while provider packages are distributed via NuGet.
```

The "flow" of using ADO.NET at its simplest:

```csharp
// 1. Create a connection
using var conn = new SqlConnection("Server=localhost;Database=MyDb;Integrated Security=true");

// 2. Open it
await conn.OpenAsync();

// 3. Create a command with SQL
using var cmd = conn.CreateCommand();
cmd.CommandText = "SELECT Id, Name FROM Users WHERE Active = @active";
cmd.Parameters.AddWithValue("@active", true);

// 4. Execute and read results
using var reader = await cmd.ExecuteReaderAsync();
while (await reader.ReadAsync())
{
    int id = reader.GetInt32(0);       // strongly-typed access by ordinal
    string name = reader.GetString(1);
    Console.WriteLine($"{id}: {name}");
}
// 5. Connection disposed (returned to pool) when 'using' scope ends
```

```ad-note
title: ADO.NET vs Dapper vs EF Core
- **ADO.NET** — you do everything: write SQL, manage connections, map results manually
- **Dapper** — a micro-ORM that sits *on top of* ADO.NET. You still write SQL, but Dapper handles the object mapping. Internally it uses `DbConnection`, `DbCommand`, and `DbDataReader`
- **Entity Framework Core** — a full ORM. You write LINQ, EF generates SQL, tracks changes, and manages the database schema via migrations

All three ultimately use ADO.NET under the hood. Dapper and EF Core are abstractions *over* ADO.NET, not replacements for it.
```

```ad-note
title: Section Summary
- ADO.NET is the low-level, provider-based data access layer in .NET
- It is not an ORM — you write SQL and map results yourself
- Abstract base classes live in `System.Data.Common`; providers are NuGet packages
- Dapper and EF Core both build on top of ADO.NET internally
```

---

## ADO.NET vs Entity Framework

Understanding when to use raw ADO.NET vs Entity Framework (EF Core) is one of the most common architectural decisions in .NET data access.

| Aspect | ADO.NET | Entity Framework Core |
|---|---|---|
| **Level** | Low-level (write SQL yourself) | High-level (ORM, LINQ to Entities) |
| **Performance** | Faster (no abstraction overhead) | Slower (change tracking, query translation, materialization) |
| **Control** | Full control over SQL, execution plans, hints | SQL is generated — less control over exact output |
| **Learning curve** | Must know SQL well | Can start without deep SQL knowledge |
| **Object mapping** | Manual (`reader.GetString(0)`) | Automatic (POCO classes ← → tables) |
| **Change tracking** | None — you manage inserts/updates yourself | Built-in change tracker detects modifications |
| **Migrations** | Manual SQL scripts | Code-first migrations with `dotnet ef` |
| **Stored procedures** | First-class support | Supported but less ergonomic |
| **Bulk operations** | Easy with raw SQL (`INSERT INTO ... SELECT`) | Requires third-party libraries or raw SQL escape hatch |
| **Best for** | Performance-critical paths, complex queries, stored procs, bulk ops | CRUD applications, rapid development, domain-driven design |

```ad-warning
title: Common Misconception
"ADO.NET is outdated and you should always use EF Core." This is false. ADO.NET is the ==foundation== that EF Core itself runs on. For performance-critical code paths, complex reporting queries, bulk data operations, or when you need precise control over the SQL being executed, ADO.NET (or Dapper on top of it) is often the better choice. Many production systems use both: EF Core for standard CRUD and ADO.NET/Dapper for hot paths.
```

### Performance Comparison

In benchmarks, raw ADO.NET with `DbDataReader` is typically **2-5x faster** than EF Core for read operations due to:

1. **No query translation** — your SQL goes straight to the database, no LINQ-to-SQL compilation step
2. **No change tracking** — EF Core's `ChangeTracker` takes snapshots of every materialized entity
3. **No materialization overhead** — you read columns directly instead of constructing full entity graphs
4. **No identity resolution** — EF Core checks if an entity with the same key is already tracked

For write-heavy workloads, the gap widens further because EF Core generates individual `INSERT`/`UPDATE` statements while raw ADO.NET lets you use bulk operations.

```ad-note
title: Section Summary
- ADO.NET gives full SQL control and better performance; EF Core gives productivity and abstraction
- ADO.NET is 2-5x faster for reads due to no query translation, change tracking, or materialization overhead
- Many production systems use both ADO.NET and EF Core depending on the code path
- ADO.NET is not outdated — it is the foundation EF Core itself is built on
```

---

## The Two Layers of ADO.NET

ADO.NET provides two distinct programming models for interacting with data. Understanding when to use each is fundamental to effective data access.

### Connected Layer

The **connected layer** maintains an ==active, open connection to the database== for the duration of the data access operation. You execute a command and read the results in a forward-only, streaming fashion.

**Core classes:**

| Class | Role |
|---|---|
| `DbConnection` | Establishes and manages the connection to the database |
| `DbCommand` | Represents a SQL statement or stored procedure to execute |
| `DbDataReader` | Reads a forward-only, read-only stream of rows from the database |
| `DbParameter` | Represents a parameter to a `DbCommand` (for parameterized queries) |
| `DbTransaction` | Represents a database transaction |

**Characteristics:**

- Connection is open while reading data
- `DbDataReader` is **forward-only** and **read-only** — you can only move forward through rows, never backward
- Very **memory-efficient** — only one row is in memory at a time (streaming)
- **Best for**: reading large result sets, performance-critical reads, simple execute-and-forget operations

```csharp
// Connected layer example — streaming read
using var conn = new SqlConnection(connStr);
await conn.OpenAsync();

using var cmd = new SqlCommand("SELECT Id, Name, Email FROM Customers", conn);
using var reader = await cmd.ExecuteReaderAsync();

while (await reader.ReadAsync())
{
    // Only one row in memory at a time
    int id = reader.GetInt32(0);
    string name = reader.GetString(1);
    string email = reader.IsDBNull(2) ? null : reader.GetString(2); // handle NULLs!
    
    Console.WriteLine($"{id}: {name} ({email})");
}
// Connection released back to pool here
```

### Disconnected Layer

The **disconnected layer** fetches data into **in-memory structures** (`DataSet`, `DataTable`) and then ==closes the connection==. You can work with the data offline, modify it, and later synchronize changes back to the database.

**Core classes:**

| Class | Role |
|---|---|
| `DbDataAdapter` | Bridge between the database and `DataSet`/`DataTable` — fills and updates |
| `DataSet` | In-memory cache of multiple `DataTable` objects with relationships |
| `DataTable` | A single in-memory table of rows and columns |
| `DataRow` | A single row in a `DataTable` |
| `DataColumn` | Schema definition for a column in a `DataTable` |
| `DataRelation` | Defines a parent-child relationship between two `DataTable` objects |

**Characteristics:**

- Connection is open only during `Fill()` and `Update()` calls — closed between
- Data is loaded entirely into memory — suitable for small-to-medium result sets
- Supports **random access** — navigate rows in any direction, filter, sort
- Supports **change tracking** — `DataRow.RowState` tracks Added, Modified, Deleted, Unchanged
- **Best for**: data binding in UI applications, working with data offline, batch modifications

```csharp
// Disconnected layer example — DataTable
using var conn = new SqlConnection(connStr);
var adapter = new SqlDataAdapter("SELECT Id, Name, Email FROM Customers", conn);

var table = new DataTable();
adapter.Fill(table); // Opens connection, fetches data, closes connection

// Work with data offline — connection is closed
foreach (DataRow row in table.Rows)
{
    Console.WriteLine($"{row["Id"]}: {row["Name"]}");
}

// Modify data in memory
table.Rows[0]["Name"] = "Updated Name";

// Sync changes back to database
var builder = new SqlCommandBuilder(adapter);
adapter.Update(table); // Opens connection, sends UPDATE, closes connection
```

```ad-warning
title: Common Misconception
"You should always use the disconnected layer because it frees up connections." Not necessarily. `DataSet`/`DataTable` loads ALL results into memory at once, which can cause memory issues with large result sets. For reading 100,000+ rows, a `DbDataReader` (connected layer) is far more efficient because it streams one row at a time. The disconnected layer is best for small-to-medium datasets where you need random access or change tracking.
```

```ad-info
title: Modern Usage
In modern .NET development, the disconnected layer (`DataSet`/`DataTable`) is less commonly used for new code. Most developers prefer:
- **Connected layer** (`DbDataReader`) + manual mapping or Dapper for reads
- **EF Core** for change tracking and CRUD operations
- **DataTable** still appears in reporting, data import/export, and legacy codebases

However, understanding the disconnected layer is still valuable — you will encounter it in existing codebases and it remains useful for specific scenarios like bulk data manipulation and dynamic schemas.
```

```ad-note
title: Section Summary
- The **connected layer** keeps the connection open while reading (streaming, forward-only, memory-efficient)
- The **disconnected layer** loads data into memory (`DataSet`/`DataTable`), closes the connection, and works offline
- Connected is best for large result sets and performance; disconnected is best for random access and change tracking
- Modern codebases typically favor the connected layer + Dapper or EF Core over `DataSet`/`DataTable`
```

---

## Architecture Diagram

The following diagram shows how the two layers relate to each other and to the database through a data provider:

```
┌─────────────────────────────────────────────────────────┐
│                    Your C# Application                  │
│                                                         │
│  ┌─────────────────────┐  ┌──────────────────────────┐  │
│  │   Connected Layer   │  │   Disconnected Layer     │  │
│  │                     │  │                          │  │
│  │  DbConnection       │  │  DataSet / DataTable     │  │
│  │  DbCommand          │  │  DataRow / DataColumn    │  │
│  │  DbDataReader       │  │  DataRelation            │  │
│  │  DbTransaction      │  │                          │  │
│  │  DbParameter        │  │  DbDataAdapter           │  │
│  │                     │  │  DbCommandBuilder        │  │
│  └────────┬────────────┘  └────────────┬─────────────┘  │
│           │                            │                │
│           └────────────┬───────────────┘                │
│                        │                                │
│              ┌─────────▼──────────┐                     │
│              │   Data Provider    │                     │
│              │  (SqlClient, etc.) │                     │
│              └─────────┬──────────┘                     │
└────────────────────────┼────────────────────────────────┘
                         │
                         ▼
                 ┌───────────────┐
                 │   Database    │
                 │  (SQL Server, │
                 │  MySQL, etc.) │
                 └───────────────┘
```

Key observation: **both layers use the same data provider**. The `DbDataAdapter` in the disconnected layer internally uses `DbConnection`, `DbCommand`, and `DbDataReader` from the connected layer. The disconnected layer is built *on top of* the connected layer.

```ad-note
title: Section Summary
- Both layers communicate through the same data provider
- The disconnected layer (`DbDataAdapter`) internally uses connected-layer classes
- The data provider is the database-specific piece; the rest of the architecture is provider-agnostic
```

---

## Key Namespaces

| Namespace | What It Contains | When You Use It |
|---|---|---|
| `System.Data` | Core interfaces (`IDbConnection`, `IDataReader`), `DataSet`, `DataTable`, `DataRow`, enums (`CommandType`, `ConnectionState`) | Always — fundamental types |
| `System.Data.Common` | Abstract base classes (`DbConnection`, `DbCommand`, `DbDataReader`, `DbProviderFactory`) | Writing provider-agnostic code |
| `Microsoft.Data.SqlClient` | SQL Server provider (`SqlConnection`, `SqlCommand`, `SqlDataReader`) | Connecting to SQL Server |
| `MySqlConnector` | MySQL/MariaDB provider (`MySqlConnection`, `MySqlCommand`) | Connecting to MySQL or MariaDB |
| `Npgsql` | PostgreSQL provider (`NpgsqlConnection`, `NpgsqlCommand`) | Connecting to PostgreSQL |
| `Microsoft.Data.Sqlite` | SQLite provider (`SqliteConnection`, `SqliteCommand`) | Connecting to SQLite |

```ad-warning
title: System.Data.SqlClient vs Microsoft.Data.SqlClient
The old `System.Data.SqlClient` namespace is ==legacy and should not be used for new development==. Microsoft forked it into `Microsoft.Data.SqlClient` which is actively maintained, gets security patches, and supports modern features like Always Encrypted, Azure AD authentication, and configurable retry logic. If you see `using System.Data.SqlClient;` in existing code, plan to migrate it.
```

To install a provider, use NuGet:

```bash
dotnet add package Microsoft.Data.SqlClient     # SQL Server
dotnet add package MySqlConnector                # MySQL / MariaDB
dotnet add package Npgsql                        # PostgreSQL
dotnet add package Microsoft.Data.Sqlite         # SQLite
```

```ad-note
title: Section Summary
- `System.Data` and `System.Data.Common` are the core namespaces (ship with the runtime)
- Each database requires a NuGet provider package with concrete implementations
- Use `Microsoft.Data.SqlClient` (not the legacy `System.Data.SqlClient`) for SQL Server
- `MySqlConnector` is the recommended async-first provider for MySQL/MariaDB
```

---

## When to Use ADO.NET

Choose raw ADO.NET (or Dapper on top of it) over EF Core when:

- **Performance is critical** — hot paths in APIs, high-throughput batch processing, real-time systems
- **Complex SQL** — reporting queries with CTEs, window functions, PIVOT, or database-specific syntax that LINQ can't express cleanly
- **Stored procedures** — ADO.NET has first-class support; EF Core's stored proc support is more limited
- **Bulk operations** — `INSERT INTO ... SELECT`, `BULK INSERT`, `SqlBulkCopy` — no EF Core equivalent without third-party libraries
- **Dynamic schemas** — when you don't know the table structure at compile time, `DataTable` can represent any schema
- **Legacy system integration** — many existing .NET codebases use ADO.NET; you need to understand it to maintain them
- **Minimal dependency footprint** — ADO.NET is in the BCL; EF Core adds significant dependencies

Choose EF Core when:

- Rapid development of CRUD-heavy applications
- You want automatic change tracking and migrations
- Your team is more comfortable with LINQ than SQL
- Domain-driven design with rich entity models

```ad-note
title: Section Summary
- Use ADO.NET for performance-critical paths, complex SQL, stored procs, and bulk operations
- Use EF Core for rapid CRUD development, change tracking, and when LINQ expressiveness suffices
- Many real-world systems combine both approaches in different layers
```

---

## Comprehensive Summary

```ad-important
title: Key Takeaways
**ADO.NET** is the low-level, provider-based data access technology in .NET. It is the ==foundation== that higher-level tools like Dapper and Entity Framework Core build upon.

**Architecture**: ADO.NET has two layers:
1. **Connected Layer** — `DbConnection` → `DbCommand` → `DbDataReader`. Keeps the connection open, streams results forward-only. Best for performance and large datasets.
2. **Disconnected Layer** — `DbDataAdapter` → `DataSet`/`DataTable`. Loads data into memory, works offline. Best for random access and change tracking (though less common in modern code).

**Data Providers**: Each database has a provider package (NuGet) that implements the abstract `DbConnection`, `DbCommand`, etc. for that specific database. Code against `System.Data.Common` base classes for database-agnostic code.

**When to choose ADO.NET**: Performance-critical code, complex SQL, stored procedures, bulk operations, dynamic schemas. When to choose EF Core: CRUD apps, rapid development, change tracking, LINQ-driven queries.

**Key rule**: Both layers ultimately share the same provider and connection infrastructure. Understanding ADO.NET is essential even if you primarily use EF Core, because EF Core is built on top of it.
```

---

## Related Topics

- [[Data Providers]] — deep dive into the provider model and `DbProviderFactory`
- [[Connection Strings]] — syntax, security, and `DbConnectionStringBuilder`
- [[Connection Pooling]] — how connection reuse works and why `using` is critical
- [[DbConnection]] — the connection lifecycle and best practices
- [[DbCommand]] — executing SQL statements and stored procedures
- [[DbDataReader]] — streaming forward-only reads
- [[Parameters and SQL Injection]] — parameterized queries and security
- [[Transactions in ADO.NET]] — `DbTransaction` and isolation levels
- [[DataSet and DataTable]] — the disconnected layer in detail
- [[Dapper]] — micro-ORM built on ADO.NET
- [[Entity Framework Core]] — the full ORM alternative
