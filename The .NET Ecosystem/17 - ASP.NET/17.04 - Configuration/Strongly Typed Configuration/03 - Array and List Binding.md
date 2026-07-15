---
tags:
  - csharp
  - asp-net-core
  - configuration
  - binding
  - strongly-typed
---


JSON arrays bind to `List<T>`, `T[]`, `IEnumerable<T>`, `IList<T>`, or `IReadOnlyList<T>`.

### Configuration (appsettings.json)

```json
{
  "AllowedOrigins": [
    "https://app.example.com",
    "https://admin.example.com"
  ],
  "DatabaseConnections": [
    {
      "Name": "Primary",
      "ConnectionString": "Server=db1;Database=App;..."
    },
    {
      "Name": "ReadReplica",
      "ConnectionString": "Server=db2;Database=App;..."
    }
  ]
}
```

### C# Classes

```csharp
public class CorsSettings
{
    public List<string> AllowedOrigins { get; set; } = new();
}

public class DatabaseSettings
{
    public List<DatabaseConnection> DatabaseConnections { get; set; } = new();
}

public class DatabaseConnection
{
    public string Name { get; set; } = string.Empty;
    public string ConnectionString { get; set; } = string.Empty;
}
```

### Registration and Usage

```csharp
builder.Services.Configure<CorsSettings>(builder.Configuration);
builder.Services.Configure<DatabaseSettings>(builder.Configuration);
```

```csharp
public class StartupValidator
{
    public StartupValidator(IOptions<DatabaseSettings> options)
    {
        foreach (var db in options.Value.DatabaseConnections)
        {
            Console.WriteLine($"{db.Name}: {db.ConnectionString}");
        }
    }
}
```

> [!tip] Pro Tip
> Arrays in JSON are indexed by position (0, 1, 2, ...). You can also override individual array elements using the key format `Section:0:Property` in environment variables:
> ```bash
> export DatabaseConnections__0__Name="Overridden"
> ```

### Dictionary Binding

Dictionaries also work. A JSON object with arbitrary keys maps to `Dictionary<string, T>`:

```json
{
  "FeatureFlags": {
    "DarkMode": true,
    "BetaSearch": false,
    "NewDashboard": true
  }
}
```

```csharp
public class FeatureFlagSettings
{
    public Dictionary<string, bool> FeatureFlags { get; set; } = new();
}
```

### Binder Collection Auto-Creation

If a collection property is already initialized, the binder uses the initialized value. Otherwise, the binder can create the collection instance automatically. For properties implementing any of the following interfaces, the binder creates a `List<>` of the appropriate type as the backing object:

- `IReadOnlyList<>`
- `IReadOnlyCollection<>`
- `ICollection<>`
- `IEnumerable<>`

> [!warning]
> You cannot bind to an `IEnumerable<>` property that has already been initialized, because this interface does not expose an `Add` method and the binder will not replace the backing value. You can bind to `IEnumerable<>` only if you leave its initial value `null`.

Similarly, the binder creates a `Dictionary<,>` as the backing field for properties with dictionary interfaces, as long as they use `string`, `enum`, or integer (`int`, `short`, `byte`, etc.) keys:

- `IDictionary<,>`
- `IReadOnlyDictionary<,>`

### Properties the Binder Cannot Bind

The binder cannot bind:

- **Nonpublic properties** — only public properties are discovered
- **Set-only properties** — must have a getter
- **Read-only properties with null values** — needs a setter or a pre-initialized value
- **Indexer properties** — `this[int index]` style properties are ignored

> [!summary] Section Summary
> JSON arrays bind to `List<T>`, `T[]`, or any `IEnumerable<T>` implementation. Dictionaries bind to `Dictionary<string, T>`. The binder auto-creates `List<>` or `Dictionary<,>` for interface-typed properties. It cannot bind nonpublic, set-only, null-read-only, or indexer properties. Leave `IEnumerable<>` properties as `null` for binding to work.
