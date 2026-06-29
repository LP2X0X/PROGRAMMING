---
tags:
  - csharp
  - asp-net-core
  - configuration
  - fundamentals
---


> [!info] Definition
> **`IConfiguration`** is the primary interface for reading configuration values. It represents a set of key-value pairs and supports hierarchical access through sections.

`IConfiguration` is registered in the DI container automatically and can be injected into any class:

```csharp
public class EmailService
{
    private readonly IConfiguration _config;

    public EmailService(IConfiguration config)
    {
        _config = config;
    }

    public void SendEmail()
    {
        string host = _config["Smtp:Host"];
        int port = _config.GetValue<int>("Smtp:Port");
        // ...
    }
}
```

### Key Members of IConfiguration

| Member | Description |
|---|---|
| `this[string key]` | Indexer -- returns the value as `string?` or `null` if not found |
| `GetValue<T>(string key)` | Returns the value converted to type `T` |
| `GetValue<T>(string key, T defaultValue)` | Returns the value or a default if missing |
| `GetSection(string key)` | Returns an `IConfigurationSection` for a nested group |
| `GetChildren()` | Returns all immediate child sections |
| `GetConnectionString(string name)` | Shortcut for `this[$"ConnectionStrings:{name}"]` |

> [!warning] Null Return, Not Exception
> The indexer `config["Key"]` returns `null` if the key does not exist. It does **not** throw. Always handle the possibility of missing keys, or use `GetValue<T>` with a default.

> [!summary] Section Summary
> `IConfiguration` is the central interface for reading config values. It supports indexer access, typed retrieval via `GetValue<T>`, hierarchical navigation via `GetSection`, and is automatically available through dependency injection.
