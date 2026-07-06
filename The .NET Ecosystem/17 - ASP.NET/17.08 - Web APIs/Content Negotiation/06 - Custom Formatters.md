---
tags:
  - csharp
  - asp-net-core
  - web-api
  - content-negotiation
  - serialization
---


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
