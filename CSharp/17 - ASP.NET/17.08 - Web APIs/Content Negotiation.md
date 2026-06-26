---
tags: [csharp, asp-net-core, web-api, content-negotiation, serialization]
date: 2026-06-18
---

# Content Negotiation

## Table of Contents

- [What Is Content Negotiation](#what-is-content-negotiation)
- [Default Behavior — JSON with System.Text.Json](#default-behavior--json-with-systemtextjson)
- [Adding XML Support](#adding-xml-support)
- [System.Text.Json Configuration](#systemtextjson-configuration)
- [Newtonsoft.Json — When and How](#newtonsoftjson--when-and-how)
- [Custom Formatters](#custom-formatters)
- [Produces and Consumes Attributes](#produces-and-consumes-attributes)
- [Response Compression](#response-compression)
- [Real-World Production Configuration](#real-world-production-configuration)
- [Comprehensive Summary](#comprehensive-summary)

---

## What Is Content Negotiation

**Content negotiation** is the mechanism by which ==a server selects the best representation of a resource to return to a client==, based on the client's stated preferences. In HTTP APIs, this process is driven primarily by the `Accept` request header, which tells the server what media types the client can understand.

The process works in three stages:

1. The client sends a request with an `Accept` header listing preferred media types
2. The server inspects the header and checks which **output formatters** can produce a matching media type
3. The server selects the best formatter and serializes the response accordingly

```http
GET /api/products/42 HTTP/1.1
Host: api.example.com
Accept: application/json
```

The server responds with JSON because the client requested `application/json`:

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "id": 42,
  "name": "Wireless Keyboard",
  "price": 49.99
}
```

If the same client sends `Accept: application/xml` and the server has XML formatters configured, it returns XML instead:

```http
GET /api/products/42 HTTP/1.1
Host: api.example.com
Accept: application/xml
```

```http
HTTP/1.1 200 OK
Content-Type: application/xml; charset=utf-8

<ProductDto>
  <Id>42</Id>
  <Name>Wireless Keyboard</Name>
  <Price>49.99</Price>
</ProductDto>
```

> [!ad-note]
> Content negotiation applies to both **output** (what format the response body uses) and **input** (what format the server expects for request bodies). The `Accept` header governs output negotiation. The `Content-Type` header on requests governs input negotiation.

ASP.NET Core implements content negotiation through the **ObjectResult** pipeline. When a controller action returns an object (or `Ok(object)`, `Created(...)`, etc.), the framework invokes the **output formatter selection** algorithm:

1. Collect all registered output formatters
2. Filter by those that can write the response type
3. Match against the `Accept` header's media types (in order of client preference and quality factors)
4. If no match is found, fall back to the first formatter that can write the type (by default, JSON)

```csharp
// This action returns an ObjectResult, triggering content negotiation
[HttpGet("{id}")]
public IActionResult GetProduct(int id)
{
    var product = _repository.GetById(id);
    if (product is null)
        return NotFound();

    return Ok(product); // ObjectResult — content negotiation happens here
}
```

> [!warning]
> If an action returns a specific result like `return Content("hello")` or `return Json(obj)`, content negotiation is **bypassed**. The `Json()` method always returns JSON regardless of the `Accept` header. Only `ObjectResult`-based returns participate in content negotiation.

> [!summary] Section Summary
> Content negotiation lets servers return different representations (JSON, XML, etc.) of the same resource depending on the client's `Accept` header. ASP.NET Core implements this through output formatters selected during the ObjectResult pipeline. Only ObjectResult-based responses participate in content negotiation.

---

## Default Behavior — JSON with System.Text.Json

Out of the box, ASP.NET Core ships with ==only a JSON output formatter== using **System.Text.Json** (STJ). This means that regardless of what a client requests in the `Accept` header, the server returns JSON unless additional formatters are registered.

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers(); // Registers the default JSON formatter

var app = builder.Build();
app.MapControllers();
app.Run();
```

With this setup, ASP.NET Core registers two default formatters:

| Formatter | Direction | Media Type |
|---|---|---|
| `SystemTextJsonOutputFormatter` | Output | `application/json` |
| `SystemTextJsonInputFormatter` | Input | `application/json` |

### What Happens When the Client Requests XML?

If a client sends `Accept: application/xml` but no XML formatter is registered, ASP.NET Core does **not** return `406 Not Acceptable` by default. Instead, it ==falls back to JSON==. This is a deliberate design decision to maximize compatibility.

```http
GET /api/products HTTP/1.1
Accept: application/xml

--- Response ---
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

[{"id":1,"name":"Keyboard","price":49.99}]
```

### Enabling 406 Not Acceptable

If you want strict content negotiation that returns `406` when no formatter matches the `Accept` header, configure `MvcOptions`:

```csharp
builder.Services.AddControllers(options =>
{
    options.ReturnHttpNotAcceptable = true; // Return 406 if no formatter matches
});
```

Now if a client requests `application/xml` and no XML formatter is registered:

```http
GET /api/products HTTP/1.1
Accept: application/xml

--- Response ---
HTTP/1.1 406 Not Acceptable
```

> [!tip]
> Setting `ReturnHttpNotAcceptable = true` is recommended for public APIs. It forces clients to request formats you actually support rather than silently receiving JSON when they expected something else.

### Respect Browser Accept Headers

Browsers send complex `Accept` headers like `text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8`. The `*/*` wildcard means "I accept anything," which will always match JSON. You can control this:

```csharp
builder.Services.AddControllers(options =>
{
    options.RespectBrowserAcceptHeader = true; // Don't treat browser requests specially
});
```

> [!summary] Section Summary
> ASP.NET Core defaults to JSON via System.Text.Json and falls back to JSON even when the client requests an unsupported format. Enable `ReturnHttpNotAcceptable = true` to return 406 for unsupported formats. Use `RespectBrowserAcceptHeader` to control how browser `Accept` headers are interpreted.

---

## Adding XML Support

ASP.NET Core provides built-in XML formatters that you opt into explicitly. There are two XML serializer options:

| Method | Serializer | Namespace Support | LINQ to XML |
|---|---|---|---|
| `AddXmlDataContractSerializerFormatters()` | `DataContractSerializer` | Full | No |
| `AddXmlSerializerFormatters()` | `XmlSerializer` | Limited | No |

### DataContractSerializer (Recommended)

```csharp
builder.Services.AddControllers()
    .AddXmlDataContractSerializerFormatters();
```

This registers both input and output XML formatters. Now the API responds to `Accept: application/xml`:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetProduct(int id)
    {
        var product = new ProductDto
        {
            Id = id,
            Name = "Wireless Keyboard",
            Price = 49.99m,
            Category = "Electronics"
        };

        return Ok(product); // Returns XML or JSON based on Accept header
    }
}
```

```csharp
using System.Runtime.Serialization;

[DataContract(Name = "Product", Namespace = "")]
public class ProductDto
{
    [DataMember(Order = 1)]
    public int Id { get; set; }

    [DataMember(Order = 2)]
    public string Name { get; set; } = string.Empty;

    [DataMember(Order = 3)]
    public decimal Price { get; set; }

    [DataMember(Order = 4)]
    public string Category { get; set; } = string.Empty;
}
```

> [!ad-note]
> Setting `Namespace = ""` on the `[DataContract]` attribute removes the default XML namespace, producing cleaner output. The `Order` property controls element ordering in the XML.

### XmlSerializer Alternative

```csharp
builder.Services.AddControllers()
    .AddXmlSerializerFormatters();
```

The `XmlSerializer` approach works with plain POCOs without requiring `[DataContract]` attributes but has limitations with complex object graphs and polymorphism.

### Using Both JSON and XML

With XML formatters added, the API now supports both:

```bash
# Request JSON
curl -H "Accept: application/json" https://localhost:5001/api/products/1

# Request XML
curl -H "Accept: application/xml" https://localhost:5001/api/products/1
```

### Quality Factors

Clients can express preference using **quality factors** (`q` values from 0 to 1):

```http
Accept: application/xml;q=0.9, application/json;q=1.0
```

This tells the server: "I prefer JSON (q=1.0) but will accept XML (q=0.9)." ASP.NET Core respects these quality factors during formatter selection.

> [!summary] Section Summary
> Add XML support with `AddXmlDataContractSerializerFormatters()` or `AddXmlSerializerFormatters()`. The `DataContractSerializer` is recommended for its richer namespace and ordering support. Clients can express format preference using quality factors in the `Accept` header.

---

## System.Text.Json Configuration

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

---

## Newtonsoft.Json — When and How

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

---

## Custom Formatters

When your API needs to produce or consume formats beyond JSON and XML — such as CSV, Protocol Buffers, MessagePack, or YAML — you implement **custom formatters**.

### Architecture: InputFormatter and OutputFormatter

ASP.NET Core's formatter architecture has two base classes:

| Base Class | Direction | Purpose |
|---|---|---|
| `InputFormatter` | Request body -> object | Deserialize request bodies |
| `OutputFormatter` | Object -> response body | Serialize response bodies |
| `TextInputFormatter` | Request body -> object | Text-based input (has encoding support) |
| `TextOutputFormatter` | Object -> response body | Text-based output (has encoding support) |

### Example: CSV Output Formatter

```csharp
using System.Text;
using Microsoft.AspNetCore.Mvc.Formatters;
using Microsoft.Net.Http.Headers;

public class CsvOutputFormatter : TextOutputFormatter
{
    public CsvOutputFormatter()
    {
        SupportedMediaTypes.Add(MediaTypeHeaderValue.Parse("text/csv"));
        SupportedEncodings.Add(Encoding.UTF8);
        SupportedEncodings.Add(Encoding.Unicode);
    }

    protected override bool CanWriteType(Type? type)
    {
        if (type is null) return false;

        // Support IEnumerable<T> where T has public properties
        if (type.IsGenericType)
        {
            var genericType = type.GetGenericTypeDefinition();
            if (genericType == typeof(IEnumerable<>) || genericType == typeof(List<>))
                return true;
        }

        // Also support arrays
        return type.IsArray;
    }

    public override async Task WriteResponseBodyAsync(
        OutputFormatterWriteContext context,
        Encoding selectedEncoding)
    {
        var response = context.HttpContext.Response;
        var items = (IEnumerable<object>)context.Object!;

        var sb = new StringBuilder();
        var itemType = context.ObjectType!.GetGenericArguments().FirstOrDefault()
                       ?? context.ObjectType.GetElementType()!;
        var properties = itemType.GetProperties();

        // Write header row
        sb.AppendLine(string.Join(",", properties.Select(p => p.Name)));

        // Write data rows
        foreach (var item in items)
        {
            var values = properties.Select(p =>
            {
                var value = p.GetValue(item)?.ToString() ?? "";
                // Escape values containing commas or quotes
                if (value.Contains(',') || value.Contains('"'))
                    value = $"\"{value.Replace("\"", "\"\"")}\"";
                return value;
            });
            sb.AppendLine(string.Join(",", values));
        }

        await response.WriteAsync(sb.ToString(), selectedEncoding);
    }
}
```

### Registering Custom Formatters

```csharp
builder.Services.AddControllers(options =>
{
    options.OutputFormatters.Add(new CsvOutputFormatter());
});
```

Now clients can request CSV:

```bash
curl -H "Accept: text/csv" https://localhost:5001/api/products
```

```
Id,Name,Price,Category
1,Wireless Keyboard,49.99,Electronics
2,USB-C Cable,12.99,Accessories
3,Monitor Stand,89.99,Furniture
```

### Example: CSV Input Formatter

```csharp
public class CsvInputFormatter : TextInputFormatter
{
    public CsvInputFormatter()
    {
        SupportedMediaTypes.Add(MediaTypeHeaderValue.Parse("text/csv"));
        SupportedEncodings.Add(Encoding.UTF8);
        SupportedEncodings.Add(Encoding.Unicode);
    }

    protected override bool CanReadType(Type type)
    {
        return type.IsGenericType &&
               type.GetGenericTypeDefinition() == typeof(List<>);
    }

    public override async Task<InputFormatterResult> ReadRequestBodyAsync(
        InputFormatterContext context,
        Encoding encoding)
    {
        var request = context.HttpContext.Request;
        using var reader = new StreamReader(request.Body, encoding);
        var content = await reader.ReadToEndAsync();

        var itemType = context.ModelType.GetGenericArguments()[0];
        var properties = itemType.GetProperties();
        var listType = typeof(List<>).MakeGenericType(itemType);
        var list = (System.Collections.IList)Activator.CreateInstance(listType)!;

        var lines = content.Split('\n', StringSplitOptions.RemoveEmptyEntries);
        if (lines.Length < 2) // Need at least header + 1 data row
            return await InputFormatterResult.SuccessAsync(list);

        var headers = lines[0].Trim().Split(',');

        for (int i = 1; i < lines.Length; i++)
        {
            var values = lines[i].Trim().Split(',');
            var item = Activator.CreateInstance(itemType)!;

            for (int j = 0; j < headers.Length && j < values.Length; j++)
            {
                var prop = properties.FirstOrDefault(p =>
                    p.Name.Equals(headers[j].Trim(), StringComparison.OrdinalIgnoreCase));
                if (prop is not null)
                {
                    var converted = Convert.ChangeType(values[j].Trim(), prop.PropertyType);
                    prop.SetValue(item, converted);
                }
            }

            list.Add(item);
        }

        return await InputFormatterResult.SuccessAsync(list);
    }
}
```

### Protocol Buffers Formatter

For high-performance binary serialization, use Protocol Buffers via the `protobuf-net` library:

```bash
dotnet add package protobuf-net
```

```csharp
using ProtoBuf;
using Microsoft.AspNetCore.Mvc.Formatters;

public class ProtobufOutputFormatter : OutputFormatter
{
    public ProtobufOutputFormatter()
    {
        SupportedMediaTypes.Add("application/x-protobuf");
    }

    public override async Task WriteResponseBodyAsync(OutputFormatterWriteContext context)
    {
        var response = context.HttpContext.Response;
        Serializer.Serialize(response.Body, context.Object);
        await response.Body.FlushAsync();
    }
}
```

```csharp
[ProtoContract]
public class ProductDto
{
    [ProtoMember(1)]
    public int Id { get; set; }

    [ProtoMember(2)]
    public string Name { get; set; } = string.Empty;

    [ProtoMember(3)]
    public decimal Price { get; set; }
}
```

### Controlling Formatter Order

The order of formatters matters. The first matching formatter wins:

```csharp
builder.Services.AddControllers(options =>
{
    // Insert at position 0 to make it the highest priority
    options.OutputFormatters.Insert(0, new CsvOutputFormatter());

    // Or add at the end (lowest priority)
    options.OutputFormatters.Add(new ProtobufOutputFormatter());

    // Remove a built-in formatter
    options.OutputFormatters.RemoveType<StringOutputFormatter>();
});
```

> [!summary] Section Summary
> Custom formatters extend content negotiation beyond JSON and XML. Inherit from `TextOutputFormatter` / `TextInputFormatter` for text-based formats or `OutputFormatter` / `InputFormatter` for binary formats. Register formatters in `MvcOptions`. Control priority by inserting at specific positions in the formatters list.

---

## Produces and Consumes Attributes

The `[Produces]` and `[Consumes]` attributes let you ==restrict which media types== a controller or action supports, overriding the global formatter configuration.

### [Produces] — Restricting Response Formats

Apply at the controller or action level to limit what response formats are allowed:

```csharp
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")] // All actions in this controller return JSON only
public class OrdersController : ControllerBase
{
    [HttpGet]
    public IActionResult GetOrders()
    {
        var orders = _repository.GetAll();
        return Ok(orders); // Always JSON, even if client requests XML
    }

    [HttpGet("{id}/receipt")]
    [Produces("application/pdf")] // Override: this action returns PDF
    public IActionResult GetReceipt(int id)
    {
        var pdf = _receiptService.GeneratePdf(id);
        return File(pdf, "application/pdf");
    }
}
```

Multiple media types:

```csharp
[Produces("application/json", "application/xml")]
public class ProductsController : ControllerBase
{
    // Actions negotiate between JSON and XML only (no CSV even if registered)
}
```

### [Produces] with Response Type

Combine with `typeof` for OpenAPI documentation via [[API Conventions]]:

```csharp
[HttpGet("{id}")]
[Produces("application/json")]
[ProducesResponseType(typeof(ProductDto), StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public IActionResult GetProduct(int id)
{
    var product = _repository.GetById(id);
    if (product is null)
        return NotFound();

    return Ok(product);
}
```

### [Consumes] — Restricting Request Formats

Restrict which `Content-Type` values are accepted for request bodies:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpPost]
    [Consumes("application/json")] // Only accepts JSON request bodies
    public IActionResult CreateProduct([FromBody] CreateProductRequest request)
    {
        var product = _service.Create(request);
        return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
    }

    [HttpPost("import")]
    [Consumes("text/csv")] // Only accepts CSV request bodies
    public IActionResult ImportProducts([FromBody] List<ProductDto> products)
    {
        _service.BulkImport(products);
        return Ok(new { imported = products.Count });
    }

    [HttpPost("upload")]
    [Consumes("multipart/form-data")] // Only accepts form data
    public IActionResult UploadImage(IFormFile file)
    {
        // Handle file upload
        return Ok();
    }
}
```

### What Happens When Content-Type Doesn't Match [Consumes]?

If a client sends a request with a `Content-Type` that does not match the `[Consumes]` attribute, ASP.NET Core returns `415 Unsupported Media Type`:

```http
POST /api/products HTTP/1.1
Content-Type: application/xml

<Product><Name>Widget</Name></Product>

--- Response ---
HTTP/1.1 415 Unsupported Media Type
```

> [!example]
> A common pattern is having two endpoints for the same resource — one accepting JSON and another accepting CSV — using `[Consumes]` to route to the correct action:
>
> ```csharp
> [HttpPost]
> [Consumes("application/json")]
> public IActionResult CreateProduct([FromBody] CreateProductRequest request) { ... }
>
> [HttpPost]
> [Consumes("text/csv")]
> public IActionResult CreateProductFromCsv([FromBody] List<ProductDto> products) { ... }
> ```

> [!summary] Section Summary
> `[Produces]` restricts which response formats an action can return. `[Consumes]` restricts which request body formats are accepted. Mismatched `Content-Type` headers trigger a `415 Unsupported Media Type` response. Both attributes are also consumed by OpenAPI/Swagger documentation generators.

---

## Response Compression

**Response compression** reduces the size of API responses sent over the network. While not strictly part of content negotiation, it interacts closely with it — the server compresses the serialized response body based on the client's `Accept-Encoding` header.

### How It Works

```http
GET /api/products HTTP/1.1
Accept: application/json
Accept-Encoding: gzip, deflate, br

--- Response ---
HTTP/1.1 200 OK
Content-Type: application/json
Content-Encoding: br
Vary: Accept-Encoding

[compressed body]
```

### Enabling Response Compression

```csharp
using Microsoft.AspNetCore.ResponseCompression;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = true; // Disabled by default for HTTPS (BREACH attack)
    options.Providers.Add<BrotliCompressionProvider>();
    options.Providers.Add<GzipCompressionProvider>();

    // Specify which MIME types to compress
    options.MimeTypes = ResponseCompressionDefaults.MimeTypes.Concat(new[]
    {
        "application/json",
        "application/xml",
        "text/csv",
        "application/x-protobuf"
    });
});

builder.Services.Configure<BrotliCompressionProviderOptions>(options =>
{
    options.Level = System.IO.Compression.CompressionLevel.Optimal;
});

builder.Services.Configure<GzipCompressionProviderOptions>(options =>
{
    options.Level = System.IO.Compression.CompressionLevel.SmallestSize;
});

builder.Services.AddControllers();

var app = builder.Build();

// IMPORTANT: Must be before other middleware that writes responses
app.UseResponseCompression();
app.UseRouting();
app.MapControllers();

app.Run();
```

> [!warning]
> `EnableForHttps` is disabled by default because of the ==BREACH security vulnerability==, which can exploit compression over HTTPS to leak secret tokens. Enable it only if your API does not reflect user input in responses that also contain secrets (e.g., CSRF tokens). Most pure data APIs are safe.

### Compression Levels

| Level | Speed | Compression Ratio | Use Case |
|---|---|---|---|
| `Fastest` | Best | Lowest | Real-time/streaming APIs |
| `Optimal` | Balanced | Good | General-purpose APIs |
| `SmallestSize` | Slowest | Best | Bandwidth-constrained clients |

### Brotli vs Gzip

| Feature | Brotli | Gzip |
|---|---|---|
| Compression ratio | Better (10-25% smaller) | Good |
| Speed | Slower at high levels | Faster |
| Browser support | Modern browsers | Universal |
| Content-Encoding value | `br` | `gzip` |

> [!tip]
> In production, prefer **Brotli** (`br`) with `CompressionLevel.Optimal` as the primary compressor and **Gzip** as the fallback. Brotli provides significantly better compression for JSON payloads.

### Middleware Order

Response compression middleware must be placed ==before== any middleware that writes to the response body:

```csharp
app.UseResponseCompression(); // First
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();
```

> [!ad-note]
> In many production deployments, response compression is handled by the reverse proxy (Nginx, IIS, Cloudflare) rather than ASP.NET Core. This offloads CPU work from the application. If your reverse proxy already compresses, you generally do not need ASP.NET Core's response compression middleware.

> [!summary] Section Summary
> Enable response compression with `AddResponseCompression()` and `UseResponseCompression()`. Configure Brotli and Gzip providers for best coverage. Be aware of the BREACH vulnerability when enabling compression over HTTPS. In production, consider offloading compression to a reverse proxy.

---

## Real-World Production Configuration

Bringing everything together: here is a production-grade configuration for a JSON-first API with custom converters, XML fallback, response compression, and strict content negotiation.

### Program.cs — Full Configuration

```csharp
using System.IO.Compression;
using System.Text.Json;
using System.Text.Json.Serialization;
using Microsoft.AspNetCore.ResponseCompression;

var builder = WebApplication.CreateBuilder(args);

// --- Response Compression ---
builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = true;
    options.Providers.Add<BrotliCompressionProvider>();
    options.Providers.Add<GzipCompressionProvider>();
    options.MimeTypes = ResponseCompressionDefaults.MimeTypes.Concat(new[]
    {
        "application/json",
        "application/xml"
    });
});

builder.Services.Configure<BrotliCompressionProviderOptions>(options =>
    options.Level = CompressionLevel.Optimal);

builder.Services.Configure<GzipCompressionProviderOptions>(options =>
    options.Level = CompressionLevel.Fastest);

// --- Controllers with JSON and XML ---
builder.Services.AddControllers(options =>
{
    options.ReturnHttpNotAcceptable = true;  // 406 for unsupported formats
    options.RespectBrowserAcceptHeader = true;
})
.AddXmlDataContractSerializerFormatters()
.AddJsonOptions(options =>
{
    var jsonOptions = options.JsonSerializerOptions;

    // Naming
    jsonOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
    jsonOptions.DictionaryKeyPolicy = JsonNamingPolicy.CamelCase;

    // Behavior
    jsonOptions.DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull;
    jsonOptions.PropertyNameCaseInsensitive = true;
    jsonOptions.WriteIndented = false;
    jsonOptions.NumberHandling = JsonNumberHandling.AllowReadingFromString;
    jsonOptions.ReferenceHandler = ReferenceHandler.IgnoreCycles;

    // Converters
    jsonOptions.Converters.Add(new JsonStringEnumConverter(JsonNamingPolicy.CamelCase));
    jsonOptions.Converters.Add(new DateOnlyJsonConverter());
    jsonOptions.Converters.Add(new TimeOnlyJsonConverter());
    jsonOptions.Converters.Add(new TrimmingStringConverter());
});

var app = builder.Build();

app.UseResponseCompression();
app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### Custom DateOnly Converter

ASP.NET Core 7+ handles `DateOnly` natively, but if you need a custom format:

```csharp
public class DateOnlyJsonConverter : JsonConverter<DateOnly>
{
    private const string Format = "yyyy-MM-dd";

    public override DateOnly Read(ref Utf8JsonReader reader, Type typeToConvert,
        JsonSerializerOptions options)
    {
        var value = reader.GetString();
        if (string.IsNullOrEmpty(value))
            throw new JsonException("DateOnly value cannot be null or empty.");

        if (DateOnly.TryParseExact(value, Format, out var date))
            return date;

        throw new JsonException($"Unable to parse '{value}' as DateOnly with format '{Format}'.");
    }

    public override void Write(Utf8JsonWriter writer, DateOnly value,
        JsonSerializerOptions options)
    {
        writer.WriteStringValue(value.ToString(Format));
    }
}
```

### Custom TimeOnly Converter

```csharp
public class TimeOnlyJsonConverter : JsonConverter<TimeOnly>
{
    private const string Format = "HH:mm:ss";

    public override TimeOnly Read(ref Utf8JsonReader reader, Type typeToConvert,
        JsonSerializerOptions options)
    {
        var value = reader.GetString();
        if (string.IsNullOrEmpty(value))
            throw new JsonException("TimeOnly value cannot be null or empty.");

        if (TimeOnly.TryParseExact(value, Format, out var time))
            return time;

        throw new JsonException($"Unable to parse '{value}' as TimeOnly with format '{Format}'.");
    }

    public override void Write(Utf8JsonWriter writer, TimeOnly value,
        JsonSerializerOptions options)
    {
        writer.WriteStringValue(value.ToString(Format));
    }
}
```

### Trimming String Converter

A practical converter that trims whitespace from all incoming string values:

```csharp
public class TrimmingStringConverter : JsonConverter<string>
{
    public override string? Read(ref Utf8JsonReader reader, Type typeToConvert,
        JsonSerializerOptions options)
    {
        return reader.GetString()?.Trim();
    }

    public override void Write(Utf8JsonWriter writer, string value,
        JsonSerializerOptions options)
    {
        writer.WriteStringValue(value);
    }
}
```

### Enum Converter with Custom Naming

```csharp
public enum PaymentMethod
{
    CreditCard,
    DebitCard,
    BankTransfer,
    PayPal,
    CryptoCurrency
}

// Register with camelCase naming
jsonOptions.Converters.Add(new JsonStringEnumConverter(JsonNamingPolicy.CamelCase));
// "creditCard", "debitCard", "bankTransfer", etc.
```

### Production DTO Example

```csharp
public class OrderResponseDto
{
    public int Id { get; set; }

    [JsonPropertyName("order_number")]
    public string OrderNumber { get; set; } = string.Empty;

    public CustomerSummaryDto Customer { get; set; } = null!;

    public List<OrderLineDto> Lines { get; set; } = new();

    public decimal Subtotal { get; set; }
    public decimal TaxAmount { get; set; }
    public decimal Total { get; set; }

    [JsonConverter(typeof(JsonStringEnumConverter))]
    public OrderStatus Status { get; set; }

    [JsonConverter(typeof(DateOnlyJsonConverter))]
    public DateOnly OrderDate { get; set; }

    public DateTime CreatedAtUtc { get; set; }

    [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingNull)]
    public DateTime? ShippedAtUtc { get; set; }

    [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingNull)]
    public string? TrackingNumber { get; set; }

    [JsonIgnore]
    public string InternalNotes { get; set; } = string.Empty; // Never serialized
}
```

```json
{
  "id": 4521,
  "order_number": "ORD-2026-04521",
  "customer": {
    "id": 89,
    "name": "Alice Johnson",
    "email": "alice@example.com"
  },
  "lines": [
    {
      "productId": 42,
      "productName": "Wireless Keyboard",
      "quantity": 2,
      "unitPrice": 49.99,
      "lineTotal": 99.98
    }
  ],
  "subtotal": 99.98,
  "taxAmount": 8.50,
  "total": 108.48,
  "status": "shipped",
  "orderDate": "2026-06-18",
  "createdAtUtc": "2026-06-18T09:15:30Z",
  "shippedAtUtc": "2026-06-18T14:22:00Z",
  "trackingNumber": "1Z999AA10123456784"
}
```

> [!ad-note]
> Notice how `internalNotes` is completely absent from the JSON (via `[JsonIgnore]`), `shippedAtUtc` and `trackingNumber` would be omitted when null (via `WhenWritingNull`), and the enum `status` is serialized as a lowercase camelCase string.

### Testing Content Negotiation

```bash
# Test JSON (default)
curl -s https://localhost:5001/api/orders/4521 | jq

# Test XML
curl -s -H "Accept: application/xml" https://localhost:5001/api/orders/4521

# Test 406 Not Acceptable (requesting unsupported format)
curl -s -o /dev/null -w "%{http_code}" \
  -H "Accept: text/csv" https://localhost:5001/api/orders/4521
# Output: 406

# Test with Accept-Encoding (compression)
curl -s -H "Accept-Encoding: br" --compressed https://localhost:5001/api/orders/4521

# Test 415 Unsupported Media Type
curl -s -o /dev/null -w "%{http_code}" \
  -X POST -H "Content-Type: text/csv" \
  -d "name,price\nWidget,9.99" \
  https://localhost:5001/api/products
# Output: 415 (if [Consumes("application/json")] is set)
```

> [!summary] Section Summary
> A production API configuration combines strict content negotiation (`ReturnHttpNotAcceptable`), JSON with custom converters for `DateOnly`/`TimeOnly`/enums/trimming, XML as a fallback, and Brotli+Gzip response compression. Custom `JsonConverter<T>` implementations handle domain-specific serialization needs. Use `[JsonIgnore]`, `[JsonPropertyName]`, and `WhenWritingNull` for precise control over the serialized shape.

---

## Comprehensive Summary

> [!summary] Comprehensive Summary
>
> **Content negotiation** is the HTTP mechanism by which ASP.NET Core selects the appropriate response format based on the client's `Accept` header. The framework routes through registered output formatters, defaulting to JSON via System.Text.Json.
>
> **Key configuration points:**
>
> - **Default**: JSON only via `SystemTextJsonOutputFormatter`
> - **XML**: Opt in with `AddXmlDataContractSerializerFormatters()`
> - **Strict negotiation**: Enable `ReturnHttpNotAcceptable = true` to return 406 for unsupported formats
> - **System.Text.Json**: Configure via `AddJsonOptions()` for controllers; common settings include `CamelCase` naming, `WhenWritingNull` ignore condition, `JsonStringEnumConverter`, and `ReferenceHandler.IgnoreCycles`
> - **Newtonsoft.Json**: Still needed for `JObject`/LINQ to JSON and JsonPath; added via `AddNewtonsoftJson()` which replaces STJ entirely
> - **Custom formatters**: Implement `TextOutputFormatter`/`TextInputFormatter` for text formats (CSV) or `OutputFormatter`/`InputFormatter` for binary formats (Protobuf)
> - **[Produces]/[Consumes]**: Restrict allowed media types per controller or action; mismatches yield 406 or 415 respectively
> - **Response compression**: Configure Brotli and Gzip via `AddResponseCompression()`; place `UseResponseCompression()` early in the middleware pipeline
> - **Production pattern**: Combine strict negotiation, custom JSON converters (`DateOnly`, enums, trimming), XML support, and response compression for a robust API serialization layer
>
> See also: [[API Controllers]], [[Minimal APIs]], [[API Conventions]]
