---
tags:
  - csharp
  - asp-net-core
  - configuration
  - fundamentals
---


You can extend the default configuration by adding extra providers to `builder.Configuration`.

### Adding a Custom JSON File

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Configuration.AddJsonFile("custom-settings.json", optional: true, reloadOnChange: true);
```

> [!warning] Order Matters
> Custom providers added after `CreateBuilder` are registered **after** the defaults. This means values in `custom-settings.json` will override values from environment variables and command-line arguments -- which is usually not what you want. If you need the custom file to be low-priority, clear and rebuild:
> ```csharp
> builder.Configuration.Sources.Clear();
> builder.Configuration
>     .AddJsonFile("custom-settings.json", optional: true)
>     .AddJsonFile("appsettings.json", optional: false)
>     .AddJsonFile($"appsettings.{builder.Environment.EnvironmentName}.json", optional: true)
>     .AddEnvironmentVariables()
>     .AddCommandLine(args);
> ```

### Adding an In-Memory Provider

Useful for testing:

```csharp
builder.Configuration.AddInMemoryCollection(new Dictionary<string, string?>
{
    ["Smtp:Host"] = "localhost",
    ["Smtp:Port"] = "25",
    ["MaxRetries"] = "1"
});
```

### Adding an XML File

```csharp
builder.Configuration.AddXmlFile("legacy-config.xml", optional: true);
```

### Adding an INI File

```csharp
builder.Configuration.AddIniFile("app.ini", optional: true);
```

### Other Built-In Providers

| Provider | NuGet Package | Use Case |
|---|---|---|
| Azure Key Vault | `Azure.Extensions.AspNetCore.Configuration.Secrets` | Production secrets in Azure |
| AWS Systems Manager | `Amazon.Extensions.Configuration.SystemsManager` | AWS parameter store |
| Azure App Configuration | `Microsoft.Extensions.Configuration.AzureAppConfiguration` | Centralized config in Azure |
| Custom provider | Implement `IConfigurationSource` + `IConfigurationProvider` | Any custom source |

> [!summary] Section Summary
> Custom providers can be added via `builder.Configuration.AddXxxFile()` or `AddInMemoryCollection()`. Be aware that providers added after `CreateBuilder` have higher priority than the defaults. Reorder sources explicitly if you need custom priority.
