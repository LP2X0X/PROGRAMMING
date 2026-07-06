---
tags:
  - csharp
  - asp-net-core
  - web-api
  - content-negotiation
  - serialization
---


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
