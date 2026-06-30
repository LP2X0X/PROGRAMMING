---
tags:
  - csharp
  - asp-net-core
  - web-api
  - content-negotiation
  - serialization
---


While System.Text.Json is the default and recommended serializer, **Newtonsoft.Json** (Json.NET) remains necessary in certain scenarios.

### When You Still Need Newtonsoft.Json

| Scenario | STJ Support | Newtonsoft Support |
|---|---|---|
| `JObject` / LINQ to JSON | Not available | Full support |
| `JsonPath` queries | Not available | Full support |
| Polymorphic deserialization (pre-.NET 7) | Limited | Full support |
| `[JsonProperty]` with complex defaults | Partial | Full support |
| `DateFormatString` customization | Limited | Full support |
| `MissingMemberHandling` | Not available | Full support |
| Legacy codebase migration | N/A | Drop-in compatibility |

### Adding Newtonsoft.Json

Install the NuGet package:

```bash
dotnet add package Microsoft.AspNetCore.Mvc.NewtonsoftJson
```

Replace the default serializer:

```csharp
builder.Services.AddControllers()
    .AddNewtonsoftJson(options =>
    {
        options.SerializerSettings.ContractResolver = new CamelCasePropertyNamesContractResolver();
        options.SerializerSettings.NullValueHandling = NullValueHandling.Ignore;
        options.SerializerSettings.ReferenceLoopHandling = ReferenceLoopHandling.Ignore;
        options.SerializerSettings.DateFormatString = "yyyy-MM-dd";
        options.SerializerSettings.Converters.Add(new StringEnumConverter());
    });
```

> [!warning]
> Calling `AddNewtonsoftJson()` ==replaces System.Text.Json entirely== for all controller-based serialization. You cannot use both simultaneously for the same pipeline. Choose one.

### LINQ to JSON Example

One of Newtonsoft.Json's killer features is dynamic JSON manipulation:

```csharp
[HttpPost("transform")]
public IActionResult TransformPayload([FromBody] JObject payload)
{
    // Dynamic property access without a DTO
    string customerName = payload["customer"]?["name"]?.ToString() ?? "Unknown";

    // JsonPath queries
    var expensiveItems = payload.SelectTokens("$.items[?(@.price > 100)]");

    // Dynamic construction
    var response = new JObject
    {
        ["customer"] = customerName,
        ["expensiveItemCount"] = expensiveItems.Count(),
        ["processedAt"] = DateTime.UtcNow
    };

    return Ok(response);
}
```

### Polymorphic Serialization

Newtonsoft.Json handles polymorphism more naturally:

```csharp
public abstract class NotificationDto
{
    public int Id { get; set; }
    public DateTime CreatedAt { get; set; }
}

public class EmailNotificationDto : NotificationDto
{
    public string RecipientEmail { get; set; } = string.Empty;
    public string Subject { get; set; } = string.Empty;
}

public class SmsNotificationDto : NotificationDto
{
    public string PhoneNumber { get; set; } = string.Empty;
}
```

```csharp
// Newtonsoft handles this with TypeNameHandling
options.SerializerSettings.TypeNameHandling = TypeNameHandling.Auto;
```

> [!warning]
> `TypeNameHandling.Auto` and `TypeNameHandling.All` are ==security risks== because they allow deserialization of arbitrary types. In .NET 7+, prefer STJ's `[JsonDerivedType]` attribute for type-safe polymorphism.

### STJ Polymorphism in .NET 7+

```csharp
[JsonDerivedType(typeof(EmailNotificationDto), typeDiscriminator: "email")]
[JsonDerivedType(typeof(SmsNotificationDto), typeDiscriminator: "sms")]
public abstract class NotificationDto
{
    public int Id { get; set; }
    public DateTime CreatedAt { get; set; }
}
```

Produces:

```json
{
  "$type": "email",
  "id": 1,
  "createdAt": "2026-06-18T10:00:00Z",
  "recipientEmail": "alice@example.com",
  "subject": "Your order shipped"
}
```

> [!summary] Section Summary
> Use Newtonsoft.Json when you need LINQ to JSON (`JObject`), JsonPath queries, or legacy compatibility. Install `Microsoft.AspNetCore.Mvc.NewtonsoftJson` and call `AddNewtonsoftJson()`. Be aware it fully replaces STJ for controllers. For polymorphism in .NET 7+, prefer STJ's `[JsonDerivedType]` over Newtonsoft's `TypeNameHandling`.
