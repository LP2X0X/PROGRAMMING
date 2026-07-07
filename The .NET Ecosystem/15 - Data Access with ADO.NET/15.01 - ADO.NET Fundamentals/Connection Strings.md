---
tags:
  - csharp
  - ado-net
  - connection-strings
aliases:
  - ADO.NET Connection Strings
  - DbConnectionStringBuilder
---

## Connection Strings

```ad-note
title: What You'll Learn
A **connection string** is the configuration that tells ADO.NET how to connect to a database. This note covers the syntax and common parameters for major databases, how to build them safely with `DbConnectionStringBuilder`, where to store them securely, and critical security considerations. Getting connection strings right is essential — a misconfigured connection string is one of the most common causes of "it works on my machine but not in production."
```

---

## Table of Contents

- [[#Anatomy of a Connection String]]
- [[#SQL Server Connection Strings]]
- [[#MySQL and MariaDB Connection Strings]]
- [[#PostgreSQL Connection Strings]]
- [[#SQLite Connection Strings]]
- [[#Building Connection Strings Safely]]
- [[#Storing Connection Strings]]
- [[#Security Best Practices]]
- [[#Troubleshooting Common Issues]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]

---

## Anatomy of a Connection String

A connection string is a ==semicolon-delimited string of key=value pairs==:

```
Key1=Value1;Key2=Value2;Key3=Value3
```

Rules:

- Keys are **case-insensitive** (`Server` = `server` = `SERVER`)
- Values containing `;` or `=` must be wrapped in **single or double quotes**: `Password="pass;word"`
- Leading and trailing whitespace around keys and values is ignored
- Many keys have **synonyms** (e.g., `Server` = `Data Source` = `Address` = `Addr`)
- An empty value is valid: `Password=;` (empty password)
- Unknown keys are silently ignored by most providers (no error — this can be a debugging trap)

```ad-warning
title: Common Misconception
"Connection string keys are standardized across providers." They are ==not==. While many keys overlap (`Server`, `Database`), each provider has its own set of recognized keys and synonyms. A key that works in SQL Server's connection string may not work (or may mean something different) in MySQL's. Always consult the provider's documentation.
```

```ad-note
title: Section Summary
- Connection strings are semicolon-delimited key=value pairs
- Keys are case-insensitive and many have synonyms
- Each provider defines its own recognized keys — they are not standardized
- Values with special characters must be quoted
```

---

## SQL Server Connection Strings

### Common Parameters

| Key (Primary / Synonym) | Example Value | Purpose | Default |
|---|---|---|---|
| `Server` / `Data Source` | `localhost`, `.\\SQLEXPRESS`, `192.168.1.1,1433` | Server address (optionally with port after comma) | — |
| `Database` / `Initial Catalog` | `MyDatabase` | Database name | — |
| `Integrated Security` / `Trusted_Connection` | `true` | Use Windows Authentication | `false` |
| `User Id` / `UID` | `sa` | SQL Authentication username | — |
| `Password` / `PWD` | `secret123` | SQL Authentication password | — |
| `Encrypt` | `true`, `false`, `Mandatory`, `Optional` | Encrypt the connection | `Mandatory` (new default!) |
| `TrustServerCertificate` | `true` | Trust self-signed certs (dev only) | `false` |
| `Connection Timeout` / `Connect Timeout` | `30` | Seconds to wait for connection | `15` |
| `Command Timeout` | `30` | Default command timeout in seconds | `30` |
| `MultipleActiveResultSets` / `MARS` | `true` | Allow multiple concurrent readers | `false` |
| `Application Name` | `MyApp` | Identifies your app in SQL Server logs | `.NET SqlClient Data Provider` |
| `Max Pool Size` | `100` | Max connections in the pool | `100` |
| `Min Pool Size` | `5` | Min connections kept open | `0` |
| `Pooling` | `true` | Enable connection pooling | `true` |
| `MultiSubnetFailover` | `true` | For Always On Availability Groups | `false` |

### Examples

```csharp
// Windows Authentication (Integrated Security)
"Server=localhost;Database=MyDb;Integrated Security=true;Encrypt=false"

// SQL Authentication (username + password)
"Server=localhost;Database=MyDb;User Id=sa;Password=MyP@ssw0rd;Encrypt=true;TrustServerCertificate=true"

// Named instance (note the escaped backslash in C#)
"Server=.\\SQLEXPRESS;Database=MyDb;Integrated Security=true;Encrypt=false"

// Remote server with custom port (port after comma, NOT colon)
"Server=192.168.1.100,1433;Database=MyDb;User Id=appuser;Password=secret;Encrypt=true"

// With MARS and custom timeouts
"Server=localhost;Database=MyDb;Integrated Security=true;MultipleActiveResultSets=true;Connection Timeout=30;Command Timeout=60"

// Azure SQL Database
"Server=myserver.database.windows.net;Database=MyDb;User Id=admin@myserver;Password=secret;Encrypt=true"
```

```ad-warning
title: Breaking Change — Encrypt Default
Starting with `Microsoft.Data.SqlClient` v4.0, the ==default for `Encrypt` changed from `false` to `Mandatory` (true)==. This means connections to SQL Server will fail if:
1. The server doesn't have a valid TLS certificate, AND
2. You haven't set `TrustServerCertificate=true`

For local development with self-signed certs, add `TrustServerCertificate=true`. For production, use a proper certificate.

This is the #1 cause of "A connection was successfully established with the server, but then an error occurred during the pre-login handshake" errors after upgrading the SqlClient package.
```

```ad-info
title: MARS (Multiple Active Result Sets)
By default, a SQL Server connection allows only ==one active reader at a time==. If you try to open a second `SqlDataReader` on the same connection, you'll get an error. Enabling `MultipleActiveResultSets=true` removes this restriction. However, MARS has performance overhead and subtle behavioral quirks with transactions, so only enable it when you genuinely need concurrent readers on the same connection.
```

```ad-note
title: Section Summary
- SQL Server uses `Server`, `Database`, `Integrated Security` or `User Id`/`Password`
- Port goes after a comma (not colon): `Server=host,1433`
- `Encrypt` defaults to `Mandatory` in modern SqlClient — add `TrustServerCertificate=true` for dev
- Use `Application Name` to identify your app in SQL Server monitoring tools
```

---

## MySQL and MariaDB Connection Strings

### Common Parameters (MySqlConnector)

| Key | Example | Purpose | Default |
|---|---|---|---|
| `Server` / `Host` | `localhost` | Database server | `localhost` |
| `Port` | `3306` | Server port | `3306` |
| `Database` | `mydb` | Database name | — |
| `User` / `User Id` / `Uid` | `root` | Username | — |
| `Password` / `Pwd` | `secret` | Password | — |
| `SslMode` | `Required`, `Preferred`, `None` | SSL/TLS mode | `Preferred` |
| `ConnectionTimeout` | `15` | Connection timeout (seconds) | `15` |
| `DefaultCommandTimeout` | `30` | Default command timeout (seconds) | `30` |
| `MaximumPoolSize` | `100` | Max connections in pool | `100` |
| `MinimumPoolSize` | `0` | Min connections in pool | `0` |
| `AllowLoadLocalInfile` | `true` | Allow `LOAD DATA LOCAL INFILE` | `false` |
| `CharacterSet` / `Charset` | `utf8mb4` | Character set for the connection | Server default |
| `ConvertZeroDateTime` | `true` | Convert `0000-00-00` to `DateTime.MinValue` | `false` |
| `AllowZeroDateTime` | `true` | Allow `MySqlDateTime` for zero dates | `false` |

### Examples

```csharp
// Basic connection
"Server=localhost;Port=3306;Database=mydb;User=root;Password=secret"

// With SSL and utf8mb4
"Server=db.example.com;Port=3306;Database=mydb;User=appuser;Password=secret;SslMode=Required;Charset=utf8mb4"

// With zero datetime handling (important for legacy MySQL data)
"Server=localhost;Database=mydb;User=root;Password=secret;ConvertZeroDateTime=true"
```

```ad-warning
title: Zero DateTime Trap
MySQL allows dates like `0000-00-00 00:00:00` which have no valid `DateTime` representation in C#. By default, reading such a value throws an exception. You have two options:
- `ConvertZeroDateTime=true` — converts `0000-00-00` to `DateTime.MinValue` (recommended)
- `AllowZeroDateTime=true` — returns a `MySqlDateTime` struct instead of `DateTime` (rarely useful)

If your database has legacy data with zero dates, you ==must== set one of these or face runtime exceptions.
```

```ad-note
title: Section Summary
- MySQL/MariaDB uses `Server`, `Port`, `Database`, `User`, `Password`
- Handle zero datetimes with `ConvertZeroDateTime=true` for legacy data
- Use `SslMode=Required` in production; `Charset=utf8mb4` for full Unicode support
```

---

## PostgreSQL Connection Strings

### Common Parameters (Npgsql)

| Key | Example | Purpose | Default |
|---|---|---|---|
| `Host` | `localhost` | Server address | `localhost` |
| `Port` | `5432` | Server port | `5432` |
| `Database` | `mydb` | Database name | Username |
| `Username` | `postgres` | Username | — |
| `Password` | `secret` | Password | — |
| `SSL Mode` | `Require`, `Prefer`, `Disable` | SSL/TLS mode | `Prefer` |
| `Timeout` | `15` | Connection timeout (seconds) | `15` |
| `Command Timeout` | `30` | Default command timeout (seconds) | `30` |
| `Maximum Pool Size` | `100` | Max pool size | `100` |
| `Minimum Pool Size` | `0` | Min pool size | `0` |
| `Include Error Detail` | `true` | Include parameter values in error messages | `false` |
| `Search Path` | `public,myschema` | PostgreSQL schema search path | — |

### Examples

```csharp
// Basic connection
"Host=localhost;Port=5432;Database=mydb;Username=postgres;Password=secret"

// With SSL and error details (development)
"Host=localhost;Database=mydb;Username=postgres;Password=secret;SSL Mode=Prefer;Include Error Detail=true"
```

```ad-note
title: Section Summary
- Npgsql uses `Host`, `Port`, `Database`, `Username`, `Password`
- `Include Error Detail=true` is useful for debugging but should be `false` in production (leaks parameter values)
- PostgreSQL defaults database name to the username if not specified
```

---

## SQLite Connection Strings

SQLite is file-based, so its connection strings are simpler:

| Key | Example | Purpose | Default |
|---|---|---|---|
| `Data Source` | `mydb.db`, `C:\\data\\mydb.db` | Path to the database file | — |
| `Mode` | `ReadWriteCreate`, `ReadWrite`, `ReadOnly`, `Memory` | File access mode | `ReadWriteCreate` |
| `Cache` | `Shared`, `Private` | Cache sharing mode | `Private` |
| `Password` | `secret` | Encryption password (requires SEE or SQLCipher) | — |

### Examples

```csharp
// File-based database (creates if not exists)
"Data Source=myapp.db"

// In-memory database (lost when connection closes)
"Data Source=:memory:"

// Shared in-memory database (persists across connections in same process)
"Data Source=InMemoryDb;Mode=Memory;Cache=Shared"

// Read-only
"Data Source=myapp.db;Mode=ReadOnly"

// Absolute path
"Data Source=C:\\data\\production.db"
```

```ad-info
title: In-Memory SQLite for Testing
`Data Source=:memory:` creates a database that exists only for the lifetime of the connection. Once you close the connection, all data is gone. This is extremely useful for ==unit testing== — each test gets a fresh, isolated database with zero setup/teardown cost. Use `Cache=Shared` if you need multiple connections to see the same in-memory database.
```

```ad-note
title: Section Summary
- SQLite connection strings primarily specify the `Data Source` (file path)
- `:memory:` creates an ephemeral in-memory database — excellent for testing
- Use `Mode=ReadOnly` for read-only access; `Cache=Shared` for shared in-memory databases
```

---

## Building Connection Strings Safely

==Never build connection strings by concatenating strings==, especially if any part comes from user input. Use `DbConnectionStringBuilder` or its provider-specific subclasses.

### Using SqlConnectionStringBuilder

```csharp
var builder = new SqlConnectionStringBuilder
{
    DataSource = "localhost",                        // Server
    InitialCatalog = "MyDb",                         // Database
    IntegratedSecurity = true,                       // Windows Auth
    Encrypt = SqlConnectionEncryptOption.Optional,   // Encrypt setting
    ConnectTimeout = 30,                             // Connection timeout
    ApplicationName = "MyApp",                       // App name for monitoring
    MaxPoolSize = 50,                                // Pool size
    MultipleActiveResultSets = true                  // MARS
};

string connStr = builder.ConnectionString;
// Output: "Data Source=localhost;Initial Catalog=MyDb;Integrated Security=True;..."

// You can also parse and modify existing connection strings
var parser = new SqlConnectionStringBuilder(existingConnStr);
parser.Password = "newPassword";    // modify one value
string modified = parser.ConnectionString;
```

### Using MySqlConnectionStringBuilder

```csharp
var builder = new MySqlConnectionStringBuilder
{
    Server = "localhost",
    Port = 3306,
    Database = "mydb",
    UserID = "root",
    Password = "secret",
    SslMode = MySqlSslMode.Required,
    CharacterSet = "utf8mb4",
    ConvertZeroDateTime = true
};

string connStr = builder.ConnectionString;
```

### Using NpgsqlConnectionStringBuilder

```csharp
var builder = new NpgsqlConnectionStringBuilder
{
    Host = "localhost",
    Port = 5432,
    Database = "mydb",
    Username = "postgres",
    Password = "secret",
    SslMode = SslMode.Prefer,
    IncludeErrorDetail = true   // dev only
};

string connStr = builder.ConnectionString;
```

```ad-warning
title: Connection String Injection
If you build connection strings by concatenating user input, an attacker can inject additional parameters:

```csharp
// ❌ DANGEROUS — user input can inject extra parameters
string userInput = "mydb;Integrated Security=true;";  // attacker adds params!
string connStr = $"Server=localhost;Database={userInput}";
// Result: "Server=localhost;Database=mydb;Integrated Security=true;"
// Attacker just enabled Windows Auth!

// ✅ SAFE — builder properly escapes values
var builder = new SqlConnectionStringBuilder();
builder.DataSource = "localhost";
builder.InitialCatalog = userInput;  // safely treated as a literal value
```
```

### Reading Individual Values

`DbConnectionStringBuilder` also lets you parse and inspect existing connection strings:

```csharp
var builder = new SqlConnectionStringBuilder(existingConnStr);

string server = builder.DataSource;
string database = builder.InitialCatalog;
bool encrypted = builder.Encrypt == SqlConnectionEncryptOption.Mandatory;
int timeout = builder.ConnectTimeout;

// Enumerate all key-value pairs
foreach (string key in builder.Keys)
{
    Console.WriteLine($"{key} = {builder[key]}");
}
```

```ad-note
title: Section Summary
- Always use `DbConnectionStringBuilder` (or provider-specific subclasses) to build and parse connection strings
- String concatenation is vulnerable to connection string injection attacks
- Builders properly escape special characters and validate values
- Builders can also parse existing connection strings for inspection or modification
```

---

## Storing Connection Strings

Connection strings must be stored securely and ==never hardcoded== in source code. Here are the recommended approaches, from simplest to most secure:

### 1. `appsettings.json` (ASP.NET Core / Generic Host)

```json
{
  "ConnectionStrings": {
    "Default": "Server=localhost;Database=MyDb;Integrated Security=true;Encrypt=false",
    "Reporting": "Server=reporting-server;Database=Reports;User Id=reader;Password=secret"
  }
}
```

```csharp
// Read from configuration
var builder = WebApplication.CreateBuilder(args);
string connStr = builder.Configuration.GetConnectionString("Default")!;

// Or via IConfiguration injection
public class UserService
{
    private readonly string _connStr;

    public UserService(IConfiguration config)
    {
        _connStr = config.GetConnectionString("Default")
            ?? throw new InvalidOperationException("Connection string 'Default' not found.");
    }
}
```

```ad-warning
title: appsettings.json is Not Secure for Secrets
`appsettings.json` is committed to source control by default. ==Never put production passwords in `appsettings.json`==. Use it for non-sensitive settings (like server names with Windows Auth) and override secrets using environment variables or user secrets.
```

### 2. Environment Variables

```bash
# Set environment variable (OS-level, CI/CD pipeline, Docker, etc.)
set ConnectionStrings__Default=Server=prod-server;Database=MyDb;User Id=app;Password=secret
```

```csharp
// .NET configuration automatically reads environment variables
// ConnectionStrings__Default maps to Configuration.GetConnectionString("Default")
// The double underscore __ acts as a section separator
```

### 3. User Secrets (Development Only)

```bash
# Initialize user secrets (once per project)
dotnet user-secrets init

# Set a connection string
dotnet user-secrets set "ConnectionStrings:Default" "Server=localhost;Database=MyDb;User Id=dev;Password=devpass"
```

```csharp
// Automatically loaded in Development environment
var builder = WebApplication.CreateBuilder(args);
// builder.Configuration already includes user secrets in Development
```

User secrets are stored in `%APPDATA%\Microsoft\UserSecrets\<user-secrets-id>\secrets.json` — outside the project directory, so they're never accidentally committed.

### 4. Azure Key Vault / AWS Secrets Manager (Production)

```csharp
// Azure Key Vault integration
var builder = WebApplication.CreateBuilder(args);
builder.Configuration.AddAzureKeyVault(
    new Uri("https://myvault.vault.azure.net/"),
    new DefaultAzureCredential());

// Secrets from Key Vault appear as regular configuration values
string connStr = builder.Configuration.GetConnectionString("Default")!;
```

### Configuration Precedence

.NET configuration sources have a priority order (last wins):

1. `appsettings.json` (lowest priority)
2. `appsettings.{Environment}.json`
3. User secrets (Development only)
4. Environment variables
5. Command-line arguments (highest priority)

This means environment variables ==override== `appsettings.json`, which is exactly what you want: define structure in `appsettings.json` and override secrets per environment.

```ad-important
title: Never Do This
```csharp
// ❌ Hardcoded connection string in source code
const string ConnStr = "Server=prod;Database=MyDb;User Id=sa;Password=P@ssw0rd!";

// ❌ Connection string in a committed config file with real passwords
// appsettings.json checked into git with production credentials
```

These are security vulnerabilities. Credentials in source control can be harvested by anyone with repository access — including former employees, compromised CI/CD pipelines, or public GitHub leaks.
```

```ad-note
title: Section Summary
- Use `appsettings.json` for structure and non-sensitive values
- Use user secrets for development credentials
- Use environment variables or secret managers (Azure Key Vault, AWS Secrets Manager) for production
- Configuration precedence ensures environment-specific overrides work automatically
- Never hardcode credentials or commit them to source control
```

---

## Security Best Practices

1. **Prefer Windows Authentication / Integrated Security** when possible — no password in the connection string at all
2. **Use least-privilege accounts** — the database user should have only the permissions the application needs (no `sa` or `root`)
3. **Encrypt connections** — set `Encrypt=true` (SQL Server) or `SslMode=Required` (MySQL/PostgreSQL) in production
4. **Rotate passwords** — use secret managers that support rotation
5. **Audit connection strings** — log which application is connecting (use `Application Name`) but never log the password
6. **Use `DbConnectionStringBuilder`** — prevents injection attacks when any part of the connection string is dynamic

```ad-note
title: Section Summary
- Prefer Integrated Security (no password needed)
- Use least-privilege database accounts
- Encrypt connections in production
- Use secret managers with rotation capabilities
```

---

## Troubleshooting Common Issues

| Error | Likely Cause | Fix |
|---|---|---|
| "A network-related or instance-specific error" | Server not running, wrong server name, firewall | Verify server name, check SQL Server is running, check firewall rules |
| "Login failed for user" | Wrong username/password or user doesn't have access to database | Verify credentials, check user has database access |
| "Pre-login handshake error" | `Encrypt=Mandatory` but server has no valid certificate | Add `TrustServerCertificate=true` (dev) or install a valid cert (prod) |
| "Timeout expired" | Server too slow to respond or pool exhausted | Increase `Connection Timeout`, check for [[Connection Pooling]] leaks |
| "Connection pool exhausted" | Not disposing connections (missing `using`) | Add `using` to all `DbConnection` instances |
| "MARS not enabled" error | Trying to open second reader on same connection | Add `MultipleActiveResultSets=true` or use separate connections |
| "Keyword not supported" | Using a key the provider doesn't recognize | Check provider-specific documentation for correct key names |

```ad-note
title: Section Summary
- Most connection errors stem from wrong server names, credential issues, or encryption mismatches
- Pool exhaustion means connections aren't being disposed — always use `using`
- Provider-specific key differences cause "keyword not supported" errors
```

---

## Comprehensive Summary

```ad-important
title: Key Takeaways
A **connection string** is a semicolon-delimited set of key=value pairs that configures how ADO.NET connects to a database. Keys are case-insensitive but ==not standardized across providers== — each provider defines its own recognized keys.

**Building safely**: Always use `DbConnectionStringBuilder` (or `SqlConnectionStringBuilder`, `MySqlConnectionStringBuilder`, etc.) instead of string concatenation to prevent connection string injection attacks.

**Storage hierarchy**:
- `appsettings.json` for structure and non-sensitive values
- **User secrets** for development credentials
- **Environment variables** or **secret managers** (Azure Key Vault) for production
- ==Never hardcode or commit credentials to source control==

**Critical gotchas**:
- `Encrypt` defaults to `Mandatory` in modern `Microsoft.Data.SqlClient` — causes errors with self-signed certs unless `TrustServerCertificate=true`
- MySQL's zero datetimes (`0000-00-00`) throw exceptions unless `ConvertZeroDateTime=true`
- SQL Server uses a comma for port (`Server=host,1433`), not a colon
- "Keyword not supported" means you're using a key from a different provider's documentation
```

---

## Related Topics

- [[ADO.NET Overview]] — architecture and the two-layer model
- [[Data Providers]] — provider-specific classes and the factory pattern
- [[Connection Pooling]] — how pool settings in connection strings affect performance
- [[DbConnection]] — using the connection string to open connections
- [[Parameters and SQL Injection]] — preventing SQL injection in queries
- [[ASP.NET Core Configuration]] — the full configuration system
