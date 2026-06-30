---
tags:
  - csharp
  - asp-net-core
  - configuration
  - binding
  - strongly-typed
---


Here is a realistic configuration demonstrating nested objects, lists, enums, and dictionaries all in one structure.

### Configuration (appsettings.json)

```json
{
  "AppSettings": {
    "ApplicationName": "OrderProcessor",
    "Version": "2.4.1",
    "Environment": "Production",
    "Retry": {
      "MaxAttempts": 3,
      "DelayMilliseconds": 1000,
      "BackoffStrategy": "Exponential"
    },
    "Endpoints": [
      {
        "Name": "OrderApi",
        "BaseUrl": "https://api.orders.example.com",
        "TimeoutSeconds": 30,
        "Headers": {
          "X-Api-Key": "abc123",
          "X-Client-Id": "order-processor"
        }
      },
      {
        "Name": "InventoryApi",
        "BaseUrl": "https://api.inventory.example.com",
        "TimeoutSeconds": 15,
        "Headers": {
          "X-Api-Key": "def456"
        }
      }
    ],
    "NotificationChannels": [ "Email", "Slack", "Sms" ]
  }
}
```

### C# Classes

```csharp
public enum BackoffStrategy
{
    Linear,
    Exponential,
    Constant
}

public class AppSettings
{
    public string ApplicationName { get; set; } = string.Empty;
    public string Version { get; set; } = string.Empty;
    public string Environment { get; set; } = string.Empty;
    public RetrySettings Retry { get; set; } = new();
    public List<EndpointSettings> Endpoints { get; set; } = new();
    public List<string> NotificationChannels { get; set; } = new();
}

public class RetrySettings
{
    public int MaxAttempts { get; set; }
    public int DelayMilliseconds { get; set; }
    public BackoffStrategy BackoffStrategy { get; set; }
}

public class EndpointSettings
{
    public string Name { get; set; } = string.Empty;
    public string BaseUrl { get; set; } = string.Empty;
    public int TimeoutSeconds { get; set; }
    public Dictionary<string, string> Headers { get; set; } = new();
}
```

### Registration

```csharp
builder.Services.Configure<AppSettings>(
    builder.Configuration.GetSection("AppSettings"));
```

> [!info] Definition
> **Enum binding** works by matching the string value in JSON to the enum member name (case-insensitive). `"Exponential"` in JSON binds to `BackoffStrategy.Exponential`. Integer values also work: `1` would bind to `BackoffStrategy.Exponential`.

> [!warning] Common Misconception
> If an enum value in JSON does not match any member name, the binding silently falls back to the **default value** (the first member or `0`). It does not throw. Always validate enum values if correctness is critical.

> [!summary] Section Summary
> Complex configurations with nested objects, lists, enums, and dictionaries all bind cleanly. Enum binding is case-insensitive by name. Always validate enum values since mismatches fail silently.
