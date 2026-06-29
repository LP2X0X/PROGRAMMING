---
tags:
  - csharp
  - asp-net-core
  - web-api
  - content-negotiation
  - serialization
---


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
