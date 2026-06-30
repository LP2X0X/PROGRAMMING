---
tags:
  - csharp
  - asp-net-core
  - configuration
  - fundamentals
---


The legacy .NET Framework approach used `web.config` (or `app.config`) with `ConfigurationManager`. ASP.NET Core replaces this entirely.

| Aspect | Legacy (`web.config`) | Modern (`IConfiguration`) |
|---|---|---|
| Format | XML | JSON (default), XML, INI, or any custom format |
| Access | `ConfigurationManager.AppSettings["Key"]` | `config["Key"]` or `config.GetValue<T>("Key")` |
| Connection strings | `ConfigurationManager.ConnectionStrings["Name"].ConnectionString` | `config.GetConnectionString("Name")` |
| Hierarchy | Flat `<appSettings>` or custom sections with boilerplate | Native hierarchy with colon delimiter |
| Environment overrides | Web.config transforms (SlowCheetah, XSLT) | Automatic with `appsettings.{Env}.json` |
| Secrets management | Encrypted sections in XML (complex) | User secrets, environment variables, vaults |
| Reload on change | Requires app restart | Built-in with `reloadOnChange: true` |
| Multiple sources | Complex, manual merging | First-class layered provider model |
| Strongly typed | Manual parsing or custom `ConfigurationSection` classes | [[Options Pattern]] with `IOptions<T>` |
| Testability | Global static -- hard to mock | Interface-based -- easy to mock |
| DI integration | None (static access) | Fully integrated |

### Legacy Code Example

```csharp
// .NET Framework -- web.config
// <appSettings>
//   <add key="MaxRetries" value="3" />
// </appSettings>

string maxRetriesStr = ConfigurationManager.AppSettings["MaxRetries"];
int maxRetries = int.Parse(maxRetriesStr);

string connStr = ConfigurationManager.ConnectionStrings["Default"].ConnectionString;
```

### Modern Equivalent

```csharp
// ASP.NET Core -- appsettings.json
// { "MaxRetries": 3, "ConnectionStrings": { "Default": "..." } }

int maxRetries = config.GetValue<int>("MaxRetries");
string? connStr = config.GetConnectionString("Default");
```

> [!warning] ConfigurationManager Still Exists in .NET 6+
> The `System.Configuration.ConfigurationManager` class was ported to .NET 6+ for backward compatibility. Do **not** use it in new ASP.NET Core projects. It reads from `app.config` (not `appsettings.json`) and does not participate in the modern configuration pipeline.

> [!summary] Section Summary
> The modern configuration system is a significant improvement over `web.config`: JSON by default, hierarchical, layered, testable, DI-friendly, and supports runtime reload. The legacy `ConfigurationManager` is a static, XML-based, flat system that should not be used in new ASP.NET Core applications.
