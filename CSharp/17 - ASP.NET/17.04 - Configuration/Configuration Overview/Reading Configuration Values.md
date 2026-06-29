---
tags:
  - csharp
  - asp-net-core
  - configuration
  - fundamentals
---


### Indexer Access

The simplest way to read a value. Always returns `string?`.

```csharp
// Flat key
string? appName = config["AppName"];

// Hierarchical key (colon-delimited)
string? smtpHost = config["Smtp:Host"];

// Deeply nested
string? defaultLogLevel = config["Logging:LogLevel:Default"];
```

### Typed Access with GetValue

`GetValue<T>` converts the string value to the specified type using `TypeConverter`.

```csharp
int maxRetries = config.GetValue<int>("MaxRetries");
bool enableCache = config.GetValue<bool>("FeatureFlags:EnableCache");
TimeSpan timeout = config.GetValue<TimeSpan>("RequestTimeout");
```

> [!tip] Always Provide a Default
> `GetValue<T>(key)` returns `default(T)` if the key is missing -- which is `0` for `int`, `false` for `bool`, etc. This can mask misconfiguration. Prefer the overload with an explicit default:
> ```csharp
> int maxRetries = config.GetValue<int>("MaxRetries", defaultValue: 3);
> ```

### Example: Reading Multiple Values

Given this `appsettings.json`:

```json
{
  "AppName": "OrderProcessor",
  "MaxRetries": 5,
  "Smtp": {
    "Host": "smtp.example.com",
    "Port": 587,
    "UseSsl": true
  }
}
```

```csharp
string? appName = config["AppName"];               // "OrderProcessor"
int maxRetries = config.GetValue<int>("MaxRetries"); // 5
string? host = config["Smtp:Host"];                  // "smtp.example.com"
int port = config.GetValue<int>("Smtp:Port");        // 587
bool useSsl = config.GetValue<bool>("Smtp:UseSsl");  // true
```

> [!summary] Section Summary
> Read values with the indexer (`config["Key"]`) for strings, or `GetValue<T>` for typed conversion. Use the colon (`:`) delimiter for hierarchical keys. Always consider providing a default value to avoid silent failures.
