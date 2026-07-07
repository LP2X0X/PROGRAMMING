---
tags:
  - csharp
  - asp-net-core
  - routing
  - endpoints
---


**.NET 7** introduced `MapGroup()` for organizing related endpoints with shared configuration.

### Basic Grouping

```csharp
var products = app.MapGroup("/api/products");

products.MapGet("/", async (IProductService service) =>
    await service.GetAllAsync());

products.MapGet("/{id:int}", async (int id, IProductService service) =>
    await service.GetByIdAsync(id));

products.MapPost("/", async (CreateProductDto dto, IProductService service) =>
    await service.CreateAsync(dto));
```

All routes in the group share the `/api/products` prefix.

### Shared Metadata on Groups

```csharp
var admin = app.MapGroup("/api/admin")
    .RequireAuthorization("AdminPolicy")
    .WithTags("Administration");

admin.MapGet("/users", GetUsers);        // Requires AdminPolicy
admin.MapGet("/settings", GetSettings);  // Requires AdminPolicy
admin.MapPost("/settings", UpdateSettings); // Requires AdminPolicy
```

### Nested Groups

```csharp
var api = app.MapGroup("/api");

var v1 = api.MapGroup("/v1");
var v2 = api.MapGroup("/v2");

v1.MapGet("/products", GetProductsV1);   // GET /api/v1/products
v2.MapGet("/products", GetProductsV2);   // GET /api/v2/products
```

### Group with Filters

```csharp
var products = app.MapGroup("/api/products")
    .AddEndpointFilterFactory((factoryContext, next) =>
    {
        return async (invocationContext) =>
        {
            // Runs before every endpoint in this group
            var logger = invocationContext.HttpContext
                .RequestServices.GetRequiredService<ILogger<Program>>();
            logger.LogInformation("Product endpoint called");

            return await next(invocationContext);
        };
    });
```

> [!ad-note] Key Insight
> `MapGroup()` is the minimal API equivalent of a controller class. It provides the same organizational benefits (shared prefix, shared metadata, logical grouping) without requiring a class. For applications that outgrow a flat list of `MapGet`/`MapPost` calls, groups are the first step toward structure.

> [!summary] Section Summary
> - `MapGroup()` (.NET 7+) organizes endpoints under a shared prefix.
> - Groups can apply shared metadata: authorization, tags, CORS policies, filters.
> - Groups can be nested for versioning or hierarchical organization.
> - Groups bring controller-like organization to minimal APIs.
