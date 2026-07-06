---
tags:
  - csharp
  - ado-net
  - data-providers
aliases:
  - ADO.NET Providers
  - DbProviderFactory
  - Database Providers
---

## Data Providers

```ad-note
title: What You'll Learn
A **data provider** is the database-specific implementation of the ADO.NET abstract classes. This note covers what providers are, the major providers for each database, the abstract base classes they implement, and the `DbProviderFactory` pattern for writing database-agnostic code. Understanding providers is essential because every ADO.NET operation flows through a provider.
```

---

## Table of Contents

- [[#What is a Data Provider?]]
- [[#Common Providers Reference]]
- [[#The Abstract Base Classes]]
- [[#The Provider Factory Pattern]]
- [[#Registering Provider Factories]]
- [[#Writing Database-Agnostic Code]]
- [[#Provider-Specific Features]]
- [[#Choosing the Right Provider]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]

---

## What is a Data Provider?

A **data provider** is a set of classes that implement the ADO.NET abstract interfaces and base classes for a ==specific database engine==. Each provider translates your C# method calls into the wire protocol that its database understands.

Every provider gives you concrete implementations of these core classes:

| Abstract Base Class | What It Does | Example (SQL Server) |
|---|---|---|
| `DbConnection` | Manages connection to the database | `SqlConnection` |
| `DbCommand` | Represents a SQL statement or stored procedure | `SqlCommand` |
| `DbDataReader` | Forward-only, read-only result stream | `SqlDataReader` |
| `DbParameter` | A parameter in a parameterized query | `SqlParameter` |
| `DbTransaction` | A database transaction | `SqlTransaction` |
| `DbDataAdapter` | Bridge between database and `DataSet`/`DataTable` | `SqlDataAdapter` |

The naming convention is consistent: the provider prefix + the base class name. For MySQL: `MySqlConnection`, `MySqlCommand`, `MySqlDataReader`, etc.

```ad-info
title: Why Separate Providers?
Each database engine has a different wire protocol, different SQL dialect, different authentication mechanisms, and different feature sets. A provider encapsulates all of this behind the standard ADO.NET interface. This means your code structure stays the same regardless of whether you're talking to SQL Server, MySQL, PostgreSQL, or SQLite — only the provider classes change.
```

```ad-note
title: Section Summary
- A data provider is a set of classes implementing ADO.NET interfaces for a specific database
- Providers follow a consistent naming convention: prefix + base class name
- Each provider handles the wire protocol, SQL dialect, and authentication for its database
- Your code structure remains the same across providers — only the concrete classes differ
```

---

## Common Providers Reference

| Database | NuGet Package | Connection Class | Namespace | Notes |
|---|---|---|---|---|
| **SQL Server** | `Microsoft.Data.SqlClient` | `SqlConnection` | `Microsoft.Data.SqlClient` | Actively maintained; replaces legacy `System.Data.SqlClient` |
| **MySQL / MariaDB** | `MySqlConnector` | `MySqlConnection` | `MySqlConnector` | Truly async, high-performance; recommended over `MySql.Data` |
| **PostgreSQL** | `Npgsql` | `NpgsqlConnection` | `Npgsql` | Feature-rich, supports COPY, LISTEN/NOTIFY, JSON |
| **SQLite** | `Microsoft.Data.Sqlite` | `SqliteConnection` | `Microsoft.Data.Sqlite` | Lightweight, file-based; great for testing and embedded |
| **Oracle** | `Oracle.ManagedDataAccess.Core` | `OracleConnection` | `Oracle.ManagedDataAccess.Client` | Official Oracle provider |
| **ODBC (generic)** | `System.Data.Odbc` | `OdbcConnection` | `System.Data.Odbc` | Generic bridge to any ODBC-compliant source |
| **OLE DB** | `System.Data.OleDb` | `OleDbConnection` | `System.Data.OleDb` | Windows-only; legacy data sources |

### Installation

```bash
# Install via .NET CLI
dotnet add package Microsoft.Data.SqlClient     # SQL Server
dotnet add package MySqlConnector                # MySQL / MariaDB
dotnet add package Npgsql                        # PostgreSQL
dotnet add package Microsoft.Data.Sqlite         # SQLite
```

```ad-warning
title: System.Data.SqlClient is Legacy
The old `System.Data.SqlClient` that shipped with .NET Framework is ==no longer actively developed==. Microsoft forked it into `Microsoft.Data.SqlClient`, which receives security patches, performance improvements, and new features (Always Encrypted, Azure AD auth, configurable retry logic). Always use `Microsoft.Data.SqlClient` for new SQL Server projects.

Migration is usually straightforward: change the NuGet package and update the `using` directive from `System.Data.SqlClient` to `Microsoft.Data.SqlClient`. The class names (`SqlConnection`, `SqlCommand`, etc.) remain the same.
```

```ad-warning
title: MySql.Data vs MySqlConnector
There are two MySQL providers:
- **`MySql.Data`** (by Oracle) — the official provider, but has a history of bugs, fake async (calls are synchronous internally), and GPL licensing concerns
- **`MySqlConnector`** — a community-maintained, ==truly asynchronous==, MIT-licensed alternative that is widely recommended

Use `MySqlConnector` for new projects. It is the provider used by Dapper and EF Core's Pomelo MySQL provider internally.
```

```ad-note
title: Section Summary
- SQL Server: `Microsoft.Data.SqlClient` (not legacy `System.Data.SqlClient`)
- MySQL/MariaDB: `MySqlConnector` (not `MySql.Data`)
- PostgreSQL: `Npgsql`; SQLite: `Microsoft.Data.Sqlite`
- Each provider is a NuGet package with a consistent class naming pattern
```

---

## The Abstract Base Classes

All providers inherit from abstract base classes in the `System.Data.Common` namespace. These base classes also implement interfaces from `System.Data`:

```
System.Data.Common                         System.Data (interfaces)
─────────────────                          ──────────────────────
DbConnection          ───implements───►    IDbConnection
DbCommand             ───implements───►    IDbCommand
DbDataReader          ───implements───►    IDataReader, IDataRecord
DbParameter           ───implements───►    IDbDataParameter
DbTransaction         ───implements───►    IDbTransaction
DbDataAdapter         ───implements───►    IDbDataAdapter
DbProviderFactory     (no interface)
```

### Inheritance Example

```csharp
// SqlConnection inheritance chain:
// SqlConnection : DbConnection : Component, IDbConnection, IAsyncDisposable

// This means SqlConnection IS-A DbConnection
// You can always use the base type:
DbConnection conn = new SqlConnection(connStr);  // ✅ valid
DbCommand cmd = new SqlCommand("SELECT 1", (SqlConnection)conn);  // need cast for provider-specific constructor
DbCommand cmd2 = conn.CreateCommand();  // ✅ better — factory method, no cast needed
```

```ad-info
title: Interfaces vs Abstract Classes
You'll see both `IDbConnection` (interface) and `DbConnection` (abstract class) in documentation. In modern .NET, ==prefer the abstract base classes== (`DbConnection`, `DbCommand`, etc.) over the interfaces. The interfaces are older (dating back to .NET 1.0) and lack async methods. The abstract classes were added in .NET 2.0 and have been expanded with async support, `ReadOnlySpan<T>` overloads, and other modern APIs.

```csharp
// ❌ Avoid — IDbConnection has no async methods
IDbConnection conn = new SqlConnection(connStr);

// ✅ Prefer — DbConnection has OpenAsync, etc.
DbConnection conn = new SqlConnection(connStr);
```
```

```ad-note
title: Section Summary
- Abstract base classes live in `System.Data.Common`; interfaces live in `System.Data`
- All concrete provider classes inherit from the abstract base classes
- Prefer `DbConnection`/`DbCommand`/`DbDataReader` (abstract classes) over `IDbConnection`/`IDbCommand`/`IDataReader` (interfaces)
- The interfaces lack async methods; the abstract classes have full async support
```

---

## The Provider Factory Pattern

**`DbProviderFactory`** is a factory class that creates provider-specific objects without your code knowing which provider it's using. This is the key to ==writing database-agnostic code==.

Each provider supplies a singleton factory:

| Provider | Factory Access |
|---|---|
| SQL Server | `SqlClientFactory.Instance` |
| MySQL | `MySqlConnectorFactory.Instance` |
| PostgreSQL | `NpgsqlFactory.Instance` |
| SQLite | `SqliteFactory.Instance` |

### Basic Usage

```csharp
// Create objects through the factory — no provider-specific types in your code
DbProviderFactory factory = SqlClientFactory.Instance;

using DbConnection conn = factory.CreateConnection()!;
conn.ConnectionString = "Server=localhost;Database=MyDb;Integrated Security=true";
await conn.OpenAsync();

using DbCommand cmd = factory.CreateCommand()!;
cmd.Connection = conn;
cmd.CommandText = "SELECT Id, Name FROM Users";

using DbDataReader reader = await cmd.ExecuteReaderAsync();
while (await reader.ReadAsync())
{
    Console.WriteLine($"{reader.GetInt32(0)}: {reader.GetString(1)}");
}
```

Notice that ==no `Sql`-prefixed types appear in the code above== (except when obtaining the factory). Swap `SqlClientFactory.Instance` for `NpgsqlFactory.Instance` and the same code works against PostgreSQL.

### Practical Pattern — Injecting the Factory

```csharp
public class UserRepository
{
    private readonly DbProviderFactory _factory;
    private readonly string _connectionString;

    // Inject the factory and connection string — no provider coupling
    public UserRepository(DbProviderFactory factory, string connectionString)
    {
        _factory = factory;
        _connectionString = connectionString;
    }

    public async Task<List<User>> GetAllUsersAsync()
    {
        using var conn = _factory.CreateConnection()!;
        conn.ConnectionString = _connectionString;
        await conn.OpenAsync();

        using var cmd = conn.CreateCommand();
        cmd.CommandText = "SELECT Id, Name, Email FROM Users";

        using var reader = await cmd.ExecuteReaderAsync();
        var users = new List<User>();
        while (await reader.ReadAsync())
        {
            users.Add(new User
            {
                Id = reader.GetInt32(0),
                Name = reader.GetString(1),
                Email = reader.IsDBNull(2) ? null : reader.GetString(2)
            });
        }
        return users;
    }
}

// Registration in DI container
services.AddSingleton<DbProviderFactory>(SqlClientFactory.Instance);
services.AddSingleton(sp => new UserRepository(
    sp.GetRequiredService<DbProviderFactory>(),
    configuration.GetConnectionString("Default")!
));
```

```ad-note
title: Section Summary
- `DbProviderFactory` creates provider-specific objects without provider-specific code
- Each provider has a singleton factory (e.g., `SqlClientFactory.Instance`)
- Inject the factory for database-agnostic repository classes
- Swap the factory to switch databases with zero code changes in your business logic
```

---

## Registering Provider Factories

For the `DbProviderFactories.GetFactory("providerName")` approach to work, providers must be registered. In .NET Core/.NET 5+, this is done manually at startup:

```csharp
// Program.cs or Startup.cs — register providers
DbProviderFactories.RegisterFactory("Microsoft.Data.SqlClient", SqlClientFactory.Instance);
DbProviderFactories.RegisterFactory("MySqlConnector", MySqlConnectorFactory.Instance);
DbProviderFactories.RegisterFactory("Npgsql", NpgsqlFactory.Instance);
```

After registration, you can resolve factories by name:

```csharp
// Resolve by invariant name — useful when the provider is configured externally
string providerName = configuration["Database:Provider"]; // e.g., "Microsoft.Data.SqlClient"
DbProviderFactory factory = DbProviderFactories.GetFactory(providerName);

using var conn = factory.CreateConnection()!;
conn.ConnectionString = configuration.GetConnectionString("Default")!;
await conn.OpenAsync();
```

This is powerful for applications that must support ==multiple database backends configured at deployment time== (e.g., a SaaS product that runs on SQL Server for some customers and PostgreSQL for others).

```ad-info
title: .NET Framework vs .NET Core
In .NET Framework, provider factories were auto-registered via `machine.config` and `app.config`. In .NET Core/.NET 5+, there is no `machine.config`, so you must register factories explicitly with `DbProviderFactories.RegisterFactory()`. This is a common migration stumbling block.
```

```ad-note
title: Section Summary
- In .NET 5+, providers must be explicitly registered with `DbProviderFactories.RegisterFactory()`
- After registration, factories can be resolved by invariant name from configuration
- This enables multi-database applications configured at deployment time
```

---

## Writing Database-Agnostic Code

Here is a complete example of a data access layer that works with any database:

```csharp
public class GenericRepository
{
    private readonly DbProviderFactory _factory;
    private readonly string _connectionString;

    public GenericRepository(DbProviderFactory factory, string connectionString)
    {
        _factory = factory;
        _connectionString = connectionString;
    }

    /// <summary>
    /// Executes a parameterized query and returns a list of results
    /// mapped by the provided function.
    /// </summary>
    public async Task<List<T>> QueryAsync<T>(
        string sql,
        Func<DbDataReader, T> map,
        params (string name, object? value)[] parameters)
    {
        using var conn = _factory.CreateConnection()!;
        conn.ConnectionString = _connectionString;
        await conn.OpenAsync();

        using var cmd = conn.CreateCommand();
        cmd.CommandText = sql;

        // Add parameters safely
        foreach (var (name, value) in parameters)
        {
            var param = cmd.CreateParameter();    // factory method — creates correct type
            param.ParameterName = name;
            param.Value = value ?? DBNull.Value;   // handle null → DBNull
            cmd.Parameters.Add(param);
        }

        using var reader = await cmd.ExecuteReaderAsync();
        var results = new List<T>();
        while (await reader.ReadAsync())
        {
            results.Add(map(reader));
        }
        return results;
    }

    /// <summary>
    /// Executes a non-query command (INSERT, UPDATE, DELETE) and returns rows affected.
    /// </summary>
    public async Task<int> ExecuteAsync(
        string sql,
        params (string name, object? value)[] parameters)
    {
        using var conn = _factory.CreateConnection()!;
        conn.ConnectionString = _connectionString;
        await conn.OpenAsync();

        using var cmd = conn.CreateCommand();
        cmd.CommandText = sql;

        foreach (var (name, value) in parameters)
        {
            var param = cmd.CreateParameter();
            param.ParameterName = name;
            param.Value = value ?? DBNull.Value;
            cmd.Parameters.Add(param);
        }

        return await cmd.ExecuteNonQueryAsync();
    }
}

// Usage — works with ANY database
var repo = new GenericRepository(SqlClientFactory.Instance, connStr);

var users = await repo.QueryAsync(
    "SELECT Id, Name FROM Users WHERE Active = @active",
    reader => new User
    {
        Id = reader.GetInt32(0),
        Name = reader.GetString(1)
    },
    ("@active", true)
);
```

```ad-warning
title: SQL Dialect Differences
While the C# code is database-agnostic, ==SQL syntax is not==. Different databases have different:
- **Parameter syntax**: SQL Server uses `@param`, MySQL uses `@param` or `?`, Oracle uses `:param`
- **String quoting**: SQL Server uses `'`, MySQL allows both `'` and `"`
- **Identifier quoting**: SQL Server uses `[brackets]`, MySQL uses `` `backticks` ``, PostgreSQL uses `"double quotes"`
- **Pagination**: SQL Server uses `OFFSET...FETCH`, MySQL uses `LIMIT`, Oracle uses `ROWNUM` (legacy) or `FETCH` (12c+)

True database portability requires abstracting SQL generation as well — which is what ORMs like EF Core do.
```

```ad-note
title: Section Summary
- Use `DbProviderFactory` and abstract base classes for database-agnostic C# code
- Create parameters via `cmd.CreateParameter()` (factory method) rather than provider-specific constructors
- Remember that SQL dialect differences still exist — C# code portability does not mean SQL portability
```

---

## Provider-Specific Features

While the common ADO.NET API covers most scenarios, each provider has unique features worth knowing:

### SQL Server (`Microsoft.Data.SqlClient`)

```csharp
// SqlBulkCopy — high-performance bulk insert
using var bulkCopy = new SqlBulkCopy(conn);
bulkCopy.DestinationTableName = "Customers";
await bulkCopy.WriteToServerAsync(dataTable);     // inserts thousands of rows in seconds

// Table-Valued Parameters — pass a table as a parameter to a stored proc
var tvp = new DataTable();
tvp.Columns.Add("Id", typeof(int));
tvp.Rows.Add(1);
tvp.Rows.Add(2);

var param = new SqlParameter("@ids", SqlDbType.Structured)
{
    TypeName = "dbo.IntList",     // must match a TABLE TYPE in the database
    Value = tvp
};
```

### MySQL / MariaDB (`MySqlConnector`)

```csharp
// MySqlBulkCopy — similar to SqlBulkCopy
var bulkCopy = new MySqlBulkCopy(conn);
bulkCopy.DestinationTableName = "Customers";
await bulkCopy.WriteToServerAsync(dataReader);

// LOAD DATA LOCAL INFILE — fastest way to load CSV data
// Must enable AllowLoadLocalInfile=true in connection string
```

### PostgreSQL (`Npgsql`)

```csharp
// Binary COPY — extremely fast bulk import
using var writer = await conn.BeginBinaryImportAsync(
    "COPY customers (name, email) FROM STDIN (FORMAT BINARY)");
await writer.StartRowAsync();
await writer.WriteAsync("John", NpgsqlDbType.Text);
await writer.WriteAsync("john@example.com", NpgsqlDbType.Text);
await writer.CompleteAsync();

// LISTEN/NOTIFY — real-time notifications from PostgreSQL
conn.Notification += (sender, args) =>
{
    Console.WriteLine($"Channel: {args.Channel}, Payload: {args.Payload}");
};
using var cmd = new NpgsqlCommand("LISTEN my_channel", conn);
await cmd.ExecuteNonQueryAsync();
```

```ad-note
title: Section Summary
- SQL Server offers `SqlBulkCopy` and table-valued parameters
- MySqlConnector offers `MySqlBulkCopy` and `LOAD DATA LOCAL INFILE`
- Npgsql offers binary COPY and LISTEN/NOTIFY for real-time events
- Use provider-specific features when performance or functionality demands it
```

---

## Choosing the Right Provider

| Scenario | Recommended Provider | Why |
|---|---|---|
| SQL Server / Azure SQL | `Microsoft.Data.SqlClient` | Official, actively maintained, Azure AD support |
| MySQL 5.7+ / MariaDB 10.2+ | `MySqlConnector` | Truly async, MIT license, better performance |
| PostgreSQL 12+ | `Npgsql` | Feature-rich, COPY support, well-maintained |
| Embedded / local / testing | `Microsoft.Data.Sqlite` | Zero-config, file-based, fast for tests |
| Legacy ODBC data source | `System.Data.Odbc` | Bridge to any ODBC driver |
| Unknown at compile time | `DbProviderFactory` | Resolve provider at runtime from config |

```ad-note
title: Section Summary
- Match the provider to your database engine
- Use `DbProviderFactory` when the database is determined at deployment time
- Prefer `MySqlConnector` over `MySql.Data` for MySQL/MariaDB
```

---

## Comprehensive Summary

```ad-important
title: Key Takeaways
A **data provider** is the database-specific implementation layer in ADO.NET. It supplies concrete classes (`SqlConnection`, `MySqlCommand`, `NpgsqlDataReader`, etc.) that inherit from abstract base classes in `System.Data.Common`.

**Key providers**: `Microsoft.Data.SqlClient` (SQL Server), `MySqlConnector` (MySQL/MariaDB), `Npgsql` (PostgreSQL), `Microsoft.Data.Sqlite` (SQLite). Avoid the legacy `System.Data.SqlClient` and prefer `MySqlConnector` over Oracle's `MySql.Data`.

**Database-agnostic code**: Use `DbProviderFactory` to create connections, commands, and parameters without coupling to a specific provider. The factory pattern enables applications that support multiple databases from a single codebase.

**Abstract base classes** (`DbConnection`, `DbCommand`, `DbDataReader`) are preferred over the older interfaces (`IDbConnection`, `IDbCommand`) because they include async methods and modern API additions.

**Provider-specific features** like `SqlBulkCopy`, Npgsql's binary COPY, and MySqlConnector's bulk copy exist for scenarios where the common API isn't sufficient. Use them when performance demands it, accepting the database coupling.
```

---

## Related Topics

- [[ADO.NET Overview]] — architecture and the two-layer model
- [[Connection Strings]] — provider-specific connection string formats
- [[Connection Pooling]] — how providers manage connection pools
- [[DbConnection]] — the connection lifecycle in detail
- [[DbCommand]] — executing queries and stored procedures
- [[DbDataReader]] — streaming reads from the connected layer
- [[Parameters and SQL Injection]] — parameterized queries across providers
- [[Dapper]] — micro-ORM that works with any ADO.NET provider
