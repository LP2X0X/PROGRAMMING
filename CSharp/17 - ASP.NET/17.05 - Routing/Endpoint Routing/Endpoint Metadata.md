---
tags:
  - csharp
  - asp-net-core
  - routing
  - endpoints
---


Every endpoint in ASP.NET Core carries a **metadata collection** -- a bag of objects that middleware and filters can inspect. Metadata controls cross-cutting behavior without coupling it to the endpoint's business logic.

### Adding Metadata to Endpoints

#### On Minimal APIs (Fluent API)

```csharp
app.MapGet("/secret", () => "Top Secret")
    .RequireAuthorization()          // Adds AuthorizeAttribute metadata
    .RequireCors("AllowSpecific")    // Adds CORS policy metadata
    .WithName("GetSecret")          // Adds endpoint name (for URL generation)
    .WithTags("Security")           // Adds tags (for OpenAPI grouping)
    .WithDescription("Returns secret data")
    .WithSummary("Get Secret");
```

#### On Controllers (Attributes)

```csharp
[Authorize]                          // Authorization metadata
[EnableCors("AllowSpecific")]        // CORS metadata
[ProducesResponseType(200)]          // OpenAPI metadata
public IActionResult GetSecret() => Ok("Top Secret");
```

### Common Metadata Methods

| Method | Purpose | Works On |
|---|---|---|
| `RequireAuthorization()` | Requires authentication/authorization | Minimal APIs, groups |
| `AllowAnonymous()` | Bypasses authorization | Minimal APIs, groups |
| `RequireCors(policyName)` | Applies CORS policy | Minimal APIs, groups |
| `WithName(name)` | Sets endpoint name (for URL generation) | Minimal APIs |
| `WithTags(tags)` | Sets OpenAPI tags | Minimal APIs |
| `WithDescription(desc)` | Sets OpenAPI description | Minimal APIs |
| `WithSummary(summary)` | Sets OpenAPI summary | Minimal APIs |
| `RequireRateLimiting(policy)` | Applies rate limiting policy | Minimal APIs, groups |
| `CacheOutput(policy)` | Applies output caching | Minimal APIs, groups |
| `Produces<T>(statusCode)` | Documents response types for OpenAPI | Minimal APIs |

### How Middleware Uses Metadata

Middleware accesses metadata through the endpoint object:

```csharp
app.Use(async (context, next) =>
{
    var endpoint = context.GetEndpoint();
    if (endpoint != null)
    {
        // Check if endpoint requires authorization
        var authMetadata = endpoint.Metadata.GetMetadata<IAuthorizeData>();

        // Check if endpoint allows anonymous access
        var allowAnon = endpoint.Metadata.GetMetadata<IAllowAnonymous>();

        // Get all metadata of a specific type
        var allProduces = endpoint.Metadata.GetOrderedMetadata<IProducesResponseTypeMetadata>();
    }

    await next(context);
});
```

> [!summary] Section Summary
> - Endpoint metadata is a collection of objects attached to each endpoint.
> - Minimal APIs add metadata via fluent methods; controllers use attributes.
> - Middleware inspects metadata via `context.GetEndpoint().Metadata`.
> - Common metadata controls authorization, CORS, rate limiting, output caching, and OpenAPI documentation.
