---
tags:
  - csharp
  - asp-net-core
  - configuration
  - options-pattern
---


A **POCO (Plain Old CLR Object)** class is the foundation of the Options pattern. Its public properties must match the keys in your configuration section.

### Configuration Source (appsettings.json)

```json
{
  "Smtp": {
    "Host": "smtp.example.com",
    "Port": 587,
    "Username": "noreply@example.com",
    "Password": "s3cret!",
    "EnableSsl": true,
    "Timeout": "00:00:30"
  }
}
```

### Options POCO Class

```csharp
public class SmtpSettings
{
    public const string SectionName = "Smtp";

    public string Host { get; set; } = string.Empty;
    public int Port { get; set; } = 587;
    public string Username { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
    public bool EnableSsl { get; set; } = true;
    public TimeSpan Timeout { get; set; } = TimeSpan.FromSeconds(30);
}
```

> [!info] Definition
> An **Options class** (also called an Options POCO) is a non-abstract class with a public parameterless constructor and public read-write properties. The property names must match the configuration keys (case-insensitive by default).

### Rules for Options Classes

1. The class **must** have a public parameterless constructor
2. Properties **must** be public with both `get` and `set` accessors
3. Property names are matched to configuration keys **case-insensitively**
4. Fields are **not** bound -- only properties
5. Nested objects and collections (lists, dictionaries) are supported
6. A `const string SectionName` is a common convention to avoid repeating the section name

> [!warning] Common Misconception
> The Options class does **not** need to be a record, and you should generally avoid using `record` types for Options classes. Records generate `init`-only setters by default, which the configuration binder cannot write to in older .NET versions. Use a plain class with `{ get; set; }` properties.

### Nested and Complex Types

Configuration binding supports nested objects and collections:

```json
{
  "AppFeatures": {
    "MaxRetries": 3,
    "AllowedOrigins": [ "https://app.example.com", "https://admin.example.com" ],
    "RateLimits": {
      "RequestsPerMinute": 100,
      "BurstSize": 20
    }
  }
}
```

```csharp
public class AppFeatureFlags
{
    public const string SectionName = "AppFeatures";

    public int MaxRetries { get; set; } = 3;
    public List<string> AllowedOrigins { get; set; } = new();
    public RateLimitOptions RateLimits { get; set; } = new();
}

public class RateLimitOptions
{
    public int RequestsPerMinute { get; set; } = 60;
    public int BurstSize { get; set; } = 10;
}
```

> [!summary] Section Summary
> Options POCOs are plain C# classes with public get/set properties matching configuration keys. They support default values, nested objects, and collections. Always define a `const string SectionName` for the configuration section name.
