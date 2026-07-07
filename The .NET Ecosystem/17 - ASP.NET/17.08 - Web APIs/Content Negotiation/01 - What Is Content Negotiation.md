---
tags:
  - csharp
  - asp-net-core
  - web-api
  - content-negotiation
  - serialization
---


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
