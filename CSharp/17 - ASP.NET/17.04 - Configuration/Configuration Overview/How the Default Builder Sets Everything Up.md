---
tags:
  - csharp
  - asp-net-core
  - configuration
  - fundamentals
---


When you call `WebApplication.CreateBuilder(args)`, the framework configures the entire configuration pipeline automatically. You do not need to manually register any of the default providers.

```csharp
var builder = WebApplication.CreateBuilder(args);

// At this point, builder.Configuration already contains values from:
// 1. appsettings.json
// 2. appsettings.{Environment}.json
// 3. User secrets (if Development)
// 4. Environment variables
// 5. Command-line arguments (from 'args')

var app = builder.Build();
```

Under the hood, `CreateBuilder` does roughly this:

```csharp
// Pseudocode of what CreateBuilder sets up
var config = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
    .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true, reloadOnChange: true)
    .AddUserSecrets<Program>(optional: true)       // Development only
    .AddEnvironmentVariables()
    .AddCommandLine(args)
    .Build();
```

> [!ad-note] reloadOnChange
> The JSON providers use `reloadOnChange: true` by default. This means if you edit `appsettings.json` while the app is running, the configuration system detects the change and reloads values. This works with `IOptionsMonitor<T>` but **not** with values you read once at startup and cache.

> [!summary] Section Summary
> `WebApplication.CreateBuilder(args)` sets up all five default configuration providers automatically. You get a fully configured `IConfiguration` instance without writing any provider registration code.
