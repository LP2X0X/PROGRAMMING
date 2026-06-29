---
tags:
  - csharp
  - asp-net-core
  - minimal-apis
  - web-api
---


Minimal APIs integrate with **OpenAPI** (Swagger) to generate API documentation. The metadata methods on endpoints control how they appear in the OpenAPI specification. See [[API Conventions]] for shared conventions across your API.

### Setting Up Swagger

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add OpenAPI services
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "Product API",
        Version = "v1",
        Description = "API for managing products"
    });
});

var app = builder.Build();

// Enable Swagger middleware
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.Run();
```

### Endpoint Metadata Methods

```csharp
app.MapGet("/products/{id}", async (int id, IProductService svc) =>
{
    var product = await svc.GetByIdAsync(id);
    return product is not null
        ? Results.Ok(product)
        : Results.NotFound();
})
.WithName("GetProductById")           // Operation ID
.WithTags("Products")                 // Grouping tag
.WithDescription("Retrieves a product by its unique identifier")
.WithSummary("Get Product")           // Short summary
.Produces<Product>(200)               // 200 response with Product body
.Produces(404)                        // 404 response with no body
.Produces<ProblemDetails>(400)        // 400 response with ProblemDetails
.WithOpenApi();                       // Include in OpenAPI doc
```

### Complete Metadata Example

```csharp
var products = app.MapGroup("/api/products").WithTags("Products");

products.MapGet("/", async (
    [FromQuery] string? name,
    [FromQuery] int page,
    [FromQuery] int pageSize,
    IProductService svc) =>
{
    var result = await svc.SearchAsync(name, page, pageSize);
    return TypedResults.Ok(result);
})
.WithName("SearchProducts")
.WithSummary("Search Products")
.WithDescription("Search and paginate through the product catalog")
.WithOpenApi(operation =>
{
    operation.Parameters[0].Description = "Filter by product name (partial match)";
    operation.Parameters[1].Description = "Page number (1-based)";
    operation.Parameters[2].Description = "Items per page (max 100)";
    return operation;
});

products.MapPost("/", async (CreateProductDto dto, IProductService svc) =>
{
    var product = await svc.CreateAsync(dto);
    return TypedResults.Created($"/api/products/{product.Id}", product);
})
.WithName("CreateProduct")
.WithSummary("Create Product")
.Accepts<CreateProductDto>("application/json")
.Produces<Product>(201)
.ProducesValidationProblem()
.WithOpenApi();
```

### Built-in OpenAPI Document Generation (.NET 9+)

Starting with .NET 9, you can generate OpenAPI documents without Swashbuckle:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddOpenApi();

var app = builder.Build();

app.MapOpenApi();  // Serves OpenAPI document at /openapi/v1.json

app.Run();
```

> [!ad-note]
> .NET 9's built-in `AddOpenApi()` and `MapOpenApi()` are Microsoft's long-term replacement for the Swashbuckle dependency. For new projects targeting .NET 9+, prefer the built-in approach.

### Excluding Endpoints from OpenAPI

```csharp
// Exclude a specific endpoint from the OpenAPI document
app.MapGet("/internal/health", () => Results.Ok("Healthy"))
    .ExcludeFromDescription();
```

> [!summary] Section Summary
> Use `AddEndpointsApiExplorer()` and `AddSwaggerGen()` (or `AddOpenApi()` in .NET 9+) to generate API documentation. Decorate endpoints with `WithName()`, `WithTags()`, `Produces<T>()`, `WithSummary()`, and `WithOpenApi()` to provide rich metadata. `TypedResults` with union return types provide the best automatic metadata inference.
