---
tags:
  - csharp
  - asp-net-core
  - configuration
  - fundamentals
---


> [!info] Definition
> **`IConfigurationRoot`** extends `IConfiguration` with additional capabilities: it exposes the list of all registered providers and supports explicit reload. It is the concrete type returned by `ConfigurationBuilder.Build()`.

| Feature | `IConfiguration` | `IConfigurationRoot` |
|---|---|---|
| Read values | Yes | Yes |
| Get sections | Yes | Yes |
| List providers | No | Yes (`.Providers`) |
| Force reload | No | Yes (`.Reload()`) |
| Typical usage | Inject into services | Available at startup |

### When to Use Each

- **`IConfiguration`** -- inject this into your services. It is the abstraction your code should depend on.
- **`IConfigurationRoot`** -- use at startup when you need to inspect providers or force a reload. Do not inject it into services unless you have a specific reason.

```csharp
var builder = WebApplication.CreateBuilder(args);

// builder.Configuration is IConfigurationManager (which implements IConfigurationRoot)
IConfigurationRoot configRoot = (IConfigurationRoot)builder.Configuration;

// Inspect registered providers
foreach (IConfigurationProvider provider in configRoot.Providers)
{
    Console.WriteLine(provider.ToString());
}
```

> [!tip] Debugging Configuration Sources
> Casting to `IConfigurationRoot` and iterating `.Providers` is invaluable when troubleshooting why a value is not what you expect. You can see exactly which providers are registered and in what order.

### GetDebugView

`IConfigurationRoot` has a `GetDebugView()` extension method that dumps every key-value pair along with which provider supplied it:

```csharp
Console.WriteLine(((IConfigurationRoot)builder.Configuration).GetDebugView());
```

Output:

```
AllowedHosts=* (JsonConfigurationProvider for 'appsettings.json' (Optional))
ConnectionStrings:
  Default=Server=localhost;... (JsonConfigurationProvider for 'appsettings.Development.json' (Optional))
Logging:
  LogLevel:
    Default=Debug (JsonConfigurationProvider for 'appsettings.Development.json' (Optional))
    Microsoft=Warning (JsonConfigurationProvider for 'appsettings.json' (Optional))
```

> [!summary] Section Summary
> `IConfigurationRoot` extends `IConfiguration` with provider inspection and manual reload. Use `IConfiguration` in services; use `IConfigurationRoot` at startup for debugging. `GetDebugView()` is a powerful diagnostic tool.
