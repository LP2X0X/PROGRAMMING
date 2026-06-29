---
tags:
  - csharp
  - asp-net-core
  - web-api
  - conventions
  - openapi
---


**HATEOAS** (Hypermedia As The Engine Of Application State) is a REST constraint where ==API responses include hyperlinks that tell the client what actions are available next==. Instead of clients hardcoding URL patterns, the server guides navigation dynamically.

### Basic HATEOAS Response

```csharp
public class ProductResource
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public List<LinkDto> Links { get; set; } = [];
}

public class LinkDto
{
    public string Href { get; set; } = string.Empty;
    public string Rel { get; set; } = string.Empty;  // Relation type
    public string Method { get; set; } = string.Empty;
}
```

### Generating Links in a Controller

```csharp
[HttpGet("{id}")]
public async Task<ActionResult<ProductResource>> GetProduct(int id)
{
    var product = await _repository.GetByIdAsync(id);
    if (product is null) return NotFound();

    var resource = _mapper.Map<ProductResource>(product);
    resource.Links = new List<LinkDto>
    {
        new LinkDto
        {
            Href = Url.Action(nameof(GetProduct), new { id })!,
            Rel = "self",
            Method = "GET"
        },
        new LinkDto
        {
            Href = Url.Action(nameof(UpdateProduct), new { id })!,
            Rel = "update",
            Method = "PUT"
        },
        new LinkDto
        {
            Href = Url.Action(nameof(DeleteProduct), new { id })!,
            Rel = "delete",
            Method = "DELETE"
        }
    };

    return resource;
}
```

Response:

```json
{
  "id": 42,
  "name": "Wireless Mouse",
  "price": 29.99,
  "links": [
    { "href": "/api/products/42", "rel": "self", "method": "GET" },
    { "href": "/api/products/42", "rel": "update", "method": "PUT" },
    { "href": "/api/products/42", "rel": "delete", "method": "DELETE" }
  ]
}
```

> [!ad-note]
> HATEOAS is a full REST maturity level (Richardson Maturity Model Level 3) but is rarely implemented in practice. Most real-world APIs stop at Level 2 (HTTP verbs + resource URIs). Consider HATEOAS for public APIs with many consumers who need discoverability.

> [!summary] Section Summary
> HATEOAS adds hyperlinks to API responses that describe available actions and navigation. While a core REST principle, it is uncommon in practice. Most APIs rely on versioned documentation instead of hypermedia-driven discovery.
