---
tags:
  - csharp
  - asp-net-core
  - web-api
  - controllers
---


An **API controller** is a class that handles HTTP requests and returns data rather than HTML views. In a traditional MVC application, a controller action renders a Razor view. In an API controller, the action returns serialized data -- typically JSON -- that a frontend application, mobile app, or another service consumes.

The key distinction:

| Aspect | MVC Controller | API Controller |
|---|---|---|
| Returns | HTML views (Razor) | Data (JSON/XML) |
| Base class | `Controller` | `ControllerBase` |
| Content type | `text/html` | `application/json` |
| Consumers | Browsers | SPAs, mobile apps, services |
| View support | Yes | No |

A minimal API controller looks like this:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAll()
    {
        var products = new[]
        {
            new { Id = 1, Name = "Keyboard", Price = 49.99m },
            new { Id = 2, Name = "Mouse", Price = 29.99m }
        };

        return Ok(products);
    }
}
```

When a client sends `GET /api/products`, ASP.NET Core:

1. Routes the request to `ProductsController.GetAll()`
2. Serializes the return value to JSON (by default via `System.Text.Json`)
3. Sends the response with `Content-Type: application/json` and status `200 OK`

> [!ad-note]
> ASP.NET Core uses **content negotiation** to determine the response format. If the client sends an `Accept: application/xml` header and XML formatters are configured, the framework can serialize to XML instead. See [[Content Negotiation]] for details.

> [!summary] Section Summary
> API controllers return structured data (JSON/XML) instead of HTML views. They inherit from `ControllerBase`, are decorated with `[ApiController]`, and serve as the foundation for RESTful web services consumed by SPAs, mobile apps, and other services.
