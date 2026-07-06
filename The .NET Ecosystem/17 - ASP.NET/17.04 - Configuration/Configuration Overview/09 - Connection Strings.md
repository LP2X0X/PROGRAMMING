---
tags:
  - csharp
  - asp-net-core
  - configuration
  - fundamentals
---


Connection strings are so commonly needed that the configuration system provides a dedicated shortcut.

```csharp
string? connStr = config.GetConnectionString("Default");

// This is exactly equivalent to:
string? connStr = config["ConnectionStrings:Default"];
```

The corresponding JSON structure:

```json
{
  "ConnectionStrings": {
    "Default": "Server=localhost;Database=MyApp;Trusted_Connection=True;",
    "Reporting": "Server=reporting-db;Database=Reports;User=sa;Password=..."
  }
}
```

### Using Connection Strings in DI Registration

```csharp
var connectionString = builder.Configuration.GetConnectionString("Default")
    ?? throw new InvalidOperationException("Connection string 'Default' not found.");

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));
```

> [!tip] Throw on Missing Connection Strings
> Unlike general config values, a missing connection string almost always means the app cannot function. Fail fast with an explicit exception rather than passing `null` to your database provider.

> [!summary] Section Summary
> `GetConnectionString("Name")` is a convenience shortcut for `config["ConnectionStrings:Name"]`. Always validate that the connection string is not null before passing it to database providers.
