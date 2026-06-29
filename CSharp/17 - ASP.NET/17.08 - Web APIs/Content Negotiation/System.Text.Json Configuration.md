---
tags:
  - csharp
  - asp-net-core
  - web-api
  - content-negotiation
  - serialization
---


**System.Text.Json** (STJ) is the default JSON serializer in ASP.NET Core. It is high-performance, low-allocation, and built into the framework. Configuring it properly is essential for any production API.

### Global Configuration via JsonOptions

Configure STJ globally for all [[API Controllers]] and [[Minimal APIs]]:

```csharp
builder.Services.Configure<Microsoft.AspNetCore.Http.Json.JsonOptions>(options =>
{
    options.SerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
    options.SerializerOptions.WriteIndented = false;
    options.SerializerOptions.DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull;
    options.SerializerOptions.Converters.Add(new JsonStringEnumConverter());
});
```

> [!warning]
> There are **two** `JsonOptions` classes in ASP.NET Core. For controller-based APIs, configure `Microsoft.AspNetCore.Mvc.JsonOptions`. For [[Minimal APIs]], configure `Microsoft.AspNetCore.Http.Json.JsonOptions`. They are separate and do not share settings.

For controllers specifically:

```csharp
builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
        options.JsonSerializerOptions.WriteIndented = false;
        options.JsonSerializerOptions.DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull;
        options.JsonSerializerOptions.Converters.Add(new JsonStringEnumConverter());
        options.JsonSerializerOptions.PropertyNameCaseInsensitive = true;
    });
```

### Common Configuration Options

| Option | Default | Description |
|---|---|---|
| `PropertyNamingPolicy` | `CamelCase` | Controls JSON property name casing |
| `WriteIndented` | `false` | Pretty-print JSON output |
| `DefaultIgnoreCondition` | `Never` | Skip null or default-value properties |
| `PropertyNameCaseInsensitive` | `false` | Case-insensitive property matching on deserialization |
| `NumberHandling` | `Strict` | Allow reading numbers from strings |
| `ReferenceHandler` | `null` | Handle circular references |
| `MaxDepth` | `64` | Maximum nesting depth |

### Property Naming Policies

```csharp
// Built-in naming policies
JsonNamingPolicy.CamelCase       // "productName"
JsonNamingPolicy.SnakeCaseLower  // "product_name"
JsonNamingPolicy.SnakeCaseUpper  // "PRODUCT_NAME"
JsonNamingPolicy.KebabCaseLower  // "product-name"
JsonNamingPolicy.KebabCaseUpper  // "PRODUCT-NAME"
```

### Serialization Attributes

#### [JsonPropertyName] — Custom Property Names

```csharp
public class OrderDto
{
    [JsonPropertyName("order_id")]
    public int Id { get; set; }

    [JsonPropertyName("customer_name")]
    public string CustomerName { get; set; } = string.Empty;

    [JsonPropertyName("total_amount")]
    public decimal TotalAmount { get; set; }

    [JsonPropertyName("placed_at")]
    public DateTime PlacedAt { get; set; }
}
```

Produces:

```json
{
  "order_id": 1001,
  "customer_name": "Alice Johnson",
  "total_amount": 249.99,
  "placed_at": "2026-06-18T14:30:00Z"
}
```

#### [JsonIgnore] — Excluding Properties

```csharp
public class UserDto
{
    public int Id { get; set; }
    public string Email { get; set; } = string.Empty;

    [JsonIgnore]
    public string PasswordHash { get; set; } = string.Empty;

    [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingNull)]
    public string? MiddleName { get; set; }

    [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingDefault)]
    public int LoginCount { get; set; } // Omitted when 0
}
```

#### [JsonConverter] — Custom Converters per Property

```csharp
public class InvoiceDto
{
    public int Id { get; set; }

    [JsonConverter(typeof(DateOnlyJsonConverter))]
    public DateOnly IssueDate { get; set; }

    [JsonConverter(typeof(JsonStringEnumConverter))]
    public InvoiceStatus Status { get; set; }
}
```

### Handling Circular References

When DTOs have bidirectional navigation properties, you must handle circular references:

```csharp
builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        options.JsonSerializerOptions.ReferenceHandler = ReferenceHandler.IgnoreCycles;
    });
```

| ReferenceHandler | Behavior |
|---|---|
| `null` (default) | Throws `JsonException` on circular references |
| `ReferenceHandler.IgnoreCycles` | Replaces cycles with `null` |
| `ReferenceHandler.Preserve` | Adds `$id` / `$ref` metadata to preserve graph structure |

> [!tip]
> Use `ReferenceHandler.IgnoreCycles` for most APIs. `ReferenceHandler.Preserve` adds `$id`/`$ref` metadata that most API clients do not understand.

### Enum Serialization

By default, STJ serializes enums as integers. Use `JsonStringEnumConverter` for string names:

```csharp
public enum OrderStatus
{
    Pending,
    Processing,
    Shipped,
    Delivered,
    Cancelled
}

// Global registration
options.JsonSerializerOptions.Converters.Add(new JsonStringEnumConverter());

// Per-property
public class OrderDto
{
    [JsonConverter(typeof(JsonStringEnumConverter))]
    public OrderStatus Status { get; set; }
}
```

Without converter: `{"status": 2}` (Shipped = 2)
With converter: `{"status": "Shipped"}`

### Source Generation for Performance

For high-throughput APIs, use ==STJ source generators== to eliminate reflection:

```csharp
[JsonSerializable(typeof(ProductDto))]
[JsonSerializable(typeof(List<ProductDto>))]
[JsonSerializable(typeof(OrderDto))]
public partial class AppJsonSerializerContext : JsonSerializerContext
{
}
```

Register it:

```csharp
builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        options.JsonSerializerOptions.TypeInfoResolverChain.Insert(0, AppJsonSerializerContext.Default);
    });
```

> [!ad-note]
> Source-generated serialization improves startup time and reduces memory allocations. It is particularly valuable in containerized/serverless environments where cold start matters.

> [!summary] Section Summary
> Configure System.Text.Json globally via `AddJsonOptions()` for controllers or `Configure<JsonOptions>()` for Minimal APIs. Use attributes like `[JsonPropertyName]`, `[JsonIgnore]`, and `[JsonConverter]` for per-property control. Handle circular references with `ReferenceHandler.IgnoreCycles`. Consider source generators for high-performance scenarios.
