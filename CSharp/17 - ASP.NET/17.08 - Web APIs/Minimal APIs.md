---
tags: [csharp, asp-net-core, minimal-apis, web-api]
date: 2026-06-18
---

# Minimal APIs

## Table of Contents

- [What Are Minimal APIs](#what-are-minimal-apis)
- [Basic Syntax](#basic-syntax)
- [Route Parameters](#route-parameters)
- [Dependency Injection](#dependency-injection)
- [Request and Response Access](#request-and-response-access)
- [Parameter Binding](#parameter-binding)
- [Returning Results](#returning-results)
- [TypedResults vs Results](#typedresults-vs-results)
- [Route Groups](#route-groups)
- [Endpoint Filters](#endpoint-filters)
- [Validation](#validation)
- [OpenAPI and Swagger Integration](#openapi-and-swagger-integration)
- [File Uploads](#file-uploads)
- [Minimal APIs vs Controllers](#minimal-apis-vs-controllers)
- [Real-World Example: Complete CRUD API](#real-world-example-complete-crud-api)
- [Comprehensive Summary](#comprehensive-summary)

---

## What Are Minimal APIs

**Minimal APIs** were introduced in ==.NET 6== as a simplified approach to building HTTP APIs in ASP.NET Core. Instead of using the traditional controller-based pattern (see [[API Controllers]]), minimal APIs let you define endpoints directly in `Program.cs` (or any file) using concise lambda syntax.

The key idea: ==you map HTTP verbs to route handlers without needing controller classes, action methods, or attributes==. The entire pipeline is configured in one place using a fluent API on the `WebApplication` object.

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello, World!");

app.Run();
```

That is a complete, functional ASP.NET Core web API. No `Startup.cs`, no controller class, no `[ApiController]` attribute.

### Why Minimal APIs Exist

Before .NET 6, even a trivial API required:
- A `Startup` class with `ConfigureServices` and `Configure` methods
- At least one controller class inheriting from `ControllerBase`
- Attribute routing or convention-based routing configuration

Minimal APIs remove that ceremony. They are built on the same ASP.NET Core infrastructure (routing, middleware, DI, etc.) but expose it through a more direct programming model.

> [!ad-note]
> Minimal APIs are not a replacement for controllers. They are an alternative that is better suited for certain scenarios. Both approaches coexist in the same application if needed.

### Evolution Across .NET Versions

| .NET Version | Minimal API Feature                                  |
| ------------ | ---------------------------------------------------- |
| .NET 6       | Initial release: `MapGet`, `MapPost`, basic binding  |
| .NET 7       | Endpoint filters, route groups, `TypedResults`       |
| .NET 8       | Form binding, `[AsParameters]`, keyed DI, antiforgery|
| .NET 9       | Built-in OpenAPI document generation, `WithOpenApi()` improvements |

> [!summary] Section Summary
> Minimal APIs are a lightweight alternative to controllers for building HTTP APIs in ASP.NET Core. Introduced in .NET 6, they allow you to define endpoints directly with lambda expressions and have gained significant features in each subsequent release.

---

## Basic Syntax

Every minimal API endpoint is created by calling a `Map{Verb}` method on the `WebApplication` instance. The method takes a **route pattern** and a **handler delegate**.

### HTTP Verb Mappings

```csharp
var app = WebApplication.CreateBuilder(args).Build();

// GET request
app.MapGet("/products", () => "List of products");

// POST request
app.MapPost("/products", () => "Create a product");

// PUT request
app.MapPut("/products/{id}", (int id) => $"Update product {id}");

// DELETE request
app.MapDelete("/products/{id}", (int id) => $"Delete product {id}");

// PATCH request
app.MapPatch("/products/{id}", (int id) => $"Patch product {id}");

// Match any HTTP method
app.MapMethods("/products/report", new[] { "OPTIONS", "HEAD" }, () => "Custom methods");

// Map to all HTTP methods
app.MapFallback(() => Results.NotFound("No matching route"));

app.Run();
```

### Handler Delegate Types

The handler can be a lambda, a local function, a method group, or any `Delegate`:

```csharp
// Lambda expression
app.MapGet("/lambda", () => "From lambda");

// Named local function
app.MapGet("/local", GetMessage);
string GetMessage() => "From local function";

// Static method reference
app.MapGet("/static", ProductHandlers.GetAll);

// Instance method reference
var handler = new ProductHandler();
app.MapGet("/instance", handler.GetAll);
```

```csharp
// Separating handlers into a static class
public static class ProductHandlers
{
    public static IResult GetAll()
    {
        return Results.Ok(new[] { "Product A", "Product B" });
    }

    public static IResult GetById(int id)
    {
        return id > 0
            ? Results.Ok($"Product {id}")
            : Results.BadRequest("Invalid ID");
    }
}
```

> [!tip]
> For anything beyond a few endpoints, organize handlers into static classes grouped by domain. This keeps `Program.cs` clean while retaining the minimal API style.

### Async Handlers

Handlers can be asynchronous. The framework awaits them automatically:

```csharp
app.MapGet("/products", async (IProductRepository repo) =>
{
    var products = await repo.GetAllAsync();
    return Results.Ok(products);
});
```

> [!summary] Section Summary
> Minimal API endpoints are defined using `Map{Verb}` methods on `WebApplication`. Handlers can be lambdas, method groups, or delegates. Async handlers are fully supported. Organize handlers into separate static classes for maintainability.

---

## Route Parameters

Route parameters work the same way as in controller-based routing, but the binding happens via the handler's method parameters.

### Basic Route Parameters

```csharp
// Simple parameter
app.MapGet("/products/{id}", (int id) =>
    Results.Ok($"Product ID: {id}"));

// Multiple parameters
app.MapGet("/categories/{categoryId}/products/{productId}",
    (int categoryId, int productId) =>
        Results.Ok($"Category: {categoryId}, Product: {productId}"));

// String parameter (no conversion needed)
app.MapGet("/products/search/{name}", (string name) =>
    Results.Ok($"Searching for: {name}"));
```

### Route Constraints

You can apply **route constraints** to restrict what values match a parameter:

```csharp
// Only matches if id is an integer
app.MapGet("/products/{id:int}", (int id) =>
    Results.Ok($"Product {id}"));

// Only matches if id is a GUID
app.MapGet("/orders/{id:guid}", (Guid id) =>
    Results.Ok($"Order {id}"));

// Minimum value constraint
app.MapGet("/products/{id:int:min(1)}", (int id) =>
    Results.Ok($"Product {id}"));

// Regex constraint
app.MapGet("/products/{sku:regex(^[A-Z]{{2}}-\\d{{4}}$)}", (string sku) =>
    Results.Ok($"SKU: {sku}"));

// Length constraint
app.MapGet("/products/{code:length(5)}", (string code) =>
    Results.Ok($"Code: {code}"));
```

### Common Route Constraints

| Constraint     | Example                    | Matches                       |
| -------------- | -------------------------- | ----------------------------- |
| `int`          | `{id:int}`                 | `123`, `-1`                   |
| `bool`         | `{active:bool}`            | `true`, `false`               |
| `datetime`     | `{date:datetime}`          | `2026-06-18`                  |
| `decimal`      | `{price:decimal}`          | `49.99`                       |
| `guid`         | `{id:guid}`                | `CD2C1638-1638-72D5-...`      |
| `long`         | `{id:long}`                | `9223372036854775807`         |
| `minlength(n)` | `{name:minlength(3)}`      | Any string with 3+ chars      |
| `maxlength(n)` | `{name:maxlength(50)}`     | Any string with 50 or fewer   |
| `range(m,n)`   | `{age:range(18,120)}`      | Integer between 18 and 120    |
| `alpha`        | `{name:alpha}`             | Alphabetic characters only    |
| `required`     | `{name:required}`          | Non-empty value               |

### Catch-All Parameters

```csharp
// Catch-all: matches any remaining path segments
app.MapGet("/files/{*filePath}", (string filePath) =>
    Results.Ok($"File path: {filePath}"));
// GET /files/images/photo.jpg -> filePath = "images/photo.jpg"
```

### Optional Parameters

```csharp
// Optional route parameter with nullable type
app.MapGet("/products/{id:int?}", (int? id) =>
    id.HasValue
        ? Results.Ok($"Product {id}")
        : Results.Ok("All products"));
```

> [!summary] Section Summary
> Route parameters are declared in the route pattern with `{name}` syntax and bound to handler parameters by name. Constraints like `:int`, `:guid`, `:min(1)` restrict matches. Catch-all parameters use `{*name}`, and optional parameters use `?` with nullable types.

---

## Dependency Injection

One of the most powerful features of minimal APIs is ==automatic parameter resolution from the DI container==. If a handler parameter's type is registered in DI, the framework injects it automatically.

### Basic DI Injection

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register services
builder.Services.AddScoped<IProductService, ProductService>();
builder.Services.AddScoped<IProductRepository, ProductRepository>();
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

var app = builder.Build();

// IProductService is automatically resolved from DI
app.MapGet("/products", async (IProductService service) =>
{
    var products = await service.GetAllAsync();
    return Results.Ok(products);
});

// Multiple DI parameters are all resolved
app.MapPost("/products", async (
    CreateProductDto dto,
    IProductService service,
    ILogger<Program> logger) =>
{
    logger.LogInformation("Creating product: {Name}", dto.Name);
    var product = await service.CreateAsync(dto);
    return Results.Created($"/products/{product.Id}", product);
});

app.Run();
```

### How the Framework Resolves Parameters

The framework uses a set of rules to determine where each parameter value comes from:

1. **Route values** -- if a parameter name matches a route parameter, it binds from the route
2. **Query string** -- simple types (`string`, `int`, `bool`, etc.) not in the route bind from query string
3. **Body** -- complex types (classes, records) bind from the JSON request body (for POST/PUT/PATCH)
4. **DI services** -- if the type is registered in the DI container, it is resolved from DI
5. **Special types** -- `HttpContext`, `HttpRequest`, `HttpResponse`, `CancellationToken`, `ClaimsPrincipal` are always available

> [!warning]
> If a parameter could be ambiguous (e.g., an `int` named `id` that exists in both the route and query string), use explicit binding attributes like `[FromRoute]` or `[FromQuery]` to disambiguate.

### Explicit Service Injection with `[FromServices]`

```csharp
app.MapGet("/products", ([FromServices] IProductService service) =>
    service.GetAll());
```

> [!ad-note]
> `[FromServices]` is rarely needed because the framework already resolves DI-registered types automatically. Use it only when you need to disambiguate or make intent explicit.

### Keyed Services (.NET 8+)

```csharp
builder.Services.AddKeyedScoped<INotificationService, EmailNotificationService>("email");
builder.Services.AddKeyedScoped<INotificationService, SmsNotificationService>("sms");

app.MapPost("/notify/email", (
    [FromKeyedServices("email")] INotificationService notifier,
    NotificationDto dto) =>
{
    notifier.Send(dto);
    return Results.Ok();
});
```

> [!summary] Section Summary
> Minimal API handlers automatically resolve parameters from the DI container when their types are registered as services. The framework distinguishes between route values, query strings, request bodies, DI services, and special types. Use `[FromServices]` or `[FromKeyedServices]` when disambiguation is needed.

---

## Request and Response Access

You can access the raw HTTP request and response objects by declaring them as handler parameters.

### HttpContext, HttpRequest, HttpResponse

```csharp
// Full HttpContext access
app.MapGet("/context", (HttpContext context) =>
{
    var userAgent = context.Request.Headers.UserAgent.ToString();
    var clientIp = context.Connection.RemoteIpAddress?.ToString();
    return Results.Ok(new { userAgent, clientIp });
});

// Just the request
app.MapGet("/request-info", (HttpRequest request) =>
{
    return Results.Ok(new
    {
        Method = request.Method,
        Path = request.Path.Value,
        QueryString = request.QueryString.Value,
        ContentType = request.ContentType,
        Host = request.Host.Value
    });
});

// Direct response writing
app.MapGet("/custom-response", async (HttpResponse response) =>
{
    response.StatusCode = 200;
    response.ContentType = "text/plain";
    await response.WriteAsync("Custom response written directly");
});
```

### CancellationToken

The `CancellationToken` parameter is bound to `HttpContext.RequestAborted`, which fires when the client disconnects:

```csharp
app.MapGet("/products/export", async (
    IProductService service,
    CancellationToken cancellationToken) =>
{
    var products = await service.ExportAllAsync(cancellationToken);
    return Results.Ok(products);
});
```

### ClaimsPrincipal

Access the authenticated user directly:

```csharp
app.MapGet("/profile", (ClaimsPrincipal user) =>
{
    var userId = user.FindFirstValue(ClaimTypes.NameIdentifier);
    var email = user.FindFirstValue(ClaimTypes.Email);
    return Results.Ok(new { userId, email });
}).RequireAuthorization();
```

### Reading Request Headers

```csharp
app.MapGet("/products", (
    [FromHeader(Name = "X-Correlation-Id")] string? correlationId,
    [FromHeader(Name = "Accept-Language")] string? language) =>
{
    return Results.Ok(new { correlationId, language });
});
```

> [!summary] Section Summary
> Special types like `HttpContext`, `HttpRequest`, `HttpResponse`, `CancellationToken`, and `ClaimsPrincipal` are automatically available as handler parameters. They provide direct access to the HTTP pipeline without any explicit binding attributes.

---

## Parameter Binding

**Parameter binding** determines how values from the HTTP request are mapped to handler parameters. Minimal APIs support both implicit (convention-based) and explicit (attribute-based) binding.

### Binding Attributes

```csharp
app.MapGet("/products", (
    [FromQuery] string? name,           // From query string: ?name=Widget
    [FromQuery] int? page,              // From query string: ?page=2
    [FromQuery] int? pageSize,          // From query string: ?pageSize=10
    [FromHeader(Name = "X-Api-Key")] string apiKey,  // From header
    [FromServices] IProductService service) =>        // From DI
{
    return service.Search(name, page ?? 1, pageSize ?? 10);
});

app.MapPost("/products", (
    [FromBody] CreateProductDto dto,    // From JSON body
    [FromServices] IProductService service) =>
{
    return service.Create(dto);
});

app.MapPut("/products/{id}", (
    [FromRoute] int id,                 // From route parameter
    [FromBody] UpdateProductDto dto,
    IProductService service) =>
{
    return service.Update(id, dto);
});
```

### Binding Source Summary

| Attribute        | Source          | Default For                              |
| ---------------- | -------------- | ---------------------------------------- |
| `[FromRoute]`    | Route segment  | Parameters matching route `{name}`       |
| `[FromQuery]`    | Query string   | Simple types not in route                |
| `[FromBody]`     | Request body   | Complex types (POST/PUT/PATCH)           |
| `[FromHeader]`   | HTTP header    | Never implicit; must use attribute       |
| `[FromServices]` | DI container   | Types registered in DI                   |
| `[FromForm]`     | Form data      | When `[FromForm]` is specified (.NET 8+) |

### Implicit Binding Rules

The framework follows these rules when no explicit attribute is used:

```csharp
app.MapPost("/orders/{orderId}/items", (
    int orderId,          // [FromRoute] - matches {orderId} in the route
    string? note,         // [FromQuery] - simple type not in route -> ?note=rush
    OrderItemDto item,    // [FromBody]  - complex type -> from JSON body
    IOrderService svc,    // [FromServices] - registered in DI
    CancellationToken ct  // Special type - always available
) => { /* ... */ });
```

### The `[AsParameters]` Attribute (.NET 7+)

When you have many parameters, group them into a struct or class and use `[AsParameters]`:

```csharp
public record GetProductsRequest(
    [FromQuery] string? Name,
    [FromQuery] int Page = 1,
    [FromQuery] int PageSize = 20,
    [FromQuery] string? SortBy = "name",
    [FromHeader(Name = "X-Tenant-Id")] string? TenantId = null);

app.MapGet("/products", async (
    [AsParameters] GetProductsRequest request,
    IProductService service) =>
{
    var products = await service.SearchAsync(
        request.Name, request.Page, request.PageSize, request.SortBy);
    return Results.Ok(products);
});
```

> [!tip]
> `[AsParameters]` is especially useful when an endpoint has more than 3-4 parameters. It keeps handler signatures clean and makes the parameter contract reusable.

### Custom Binding with `TryParse` and `BindAsync`

For custom types, implement `TryParse` or `BindAsync` so the framework can bind them automatically:

```csharp
// TryParse: for simple types from route/query
public record struct Coordinate(double Latitude, double Longitude)
{
    public static bool TryParse(string? value, out Coordinate result)
    {
        result = default;
        if (value is null) return false;

        var parts = value.Split(',');
        if (parts.Length != 2) return false;

        if (double.TryParse(parts[0], out var lat) &&
            double.TryParse(parts[1], out var lng))
        {
            result = new Coordinate(lat, lng);
            return true;
        }
        return false;
    }
}

// Usage: GET /nearby?location=47.6,-122.3
app.MapGet("/nearby", (Coordinate location) =>
    Results.Ok($"Lat: {location.Latitude}, Lng: {location.Longitude}"));
```

```csharp
// BindAsync: for complex binding from the full request
public record PaginationParams(int Page, int PageSize)
{
    public static ValueTask<PaginationParams?> BindAsync(
        HttpContext context, ParameterInfo parameter)
    {
        int.TryParse(context.Request.Query["page"], out var page);
        int.TryParse(context.Request.Query["pageSize"], out var pageSize);

        var result = new PaginationParams(
            Page: page > 0 ? page : 1,
            PageSize: pageSize > 0 ? Math.Min(pageSize, 100) : 20);

        return ValueTask.FromResult<PaginationParams?>(result);
    }
}

app.MapGet("/products", (PaginationParams pagination, IProductService svc) =>
    svc.GetPaged(pagination.Page, pagination.PageSize));
```

> [!summary] Section Summary
> Parameter binding in minimal APIs uses attributes like `[FromQuery]`, `[FromRoute]`, `[FromBody]`, `[FromHeader]`, and `[FromServices]`. Implicit rules handle most cases. Use `[AsParameters]` to group many parameters into a single object. Custom types can implement `TryParse` or `BindAsync` for automatic binding.

---

## Returning Results

Minimal API handlers can return various types. The framework serializes them appropriately based on the return type.

### Implicit Returns

```csharp
// String -> text/plain 200
app.MapGet("/hello", () => "Hello, World!");

// Object -> application/json 200
app.MapGet("/product", () => new Product { Id = 1, Name = "Widget" });

// IEnumerable -> application/json 200
app.MapGet("/products", () => new List<Product>
{
    new(1, "Widget"),
    new(2, "Gadget")
});
```

### The `Results` Static Class

The `Results` class provides factory methods for common HTTP responses:

```csharp
app.MapGet("/products/{id}", async (int id, IProductService service) =>
{
    var product = await service.GetByIdAsync(id);
    return product is not null
        ? Results.Ok(product)           // 200 with JSON body
        : Results.NotFound();           // 404
});

app.MapPost("/products", async (CreateProductDto dto, IProductService service) =>
{
    var product = await service.CreateAsync(dto);
    return Results.Created(             // 201 with Location header
        $"/products/{product.Id}",
        product);
});

app.MapDelete("/products/{id}", async (int id, IProductService service) =>
{
    var deleted = await service.DeleteAsync(id);
    return deleted
        ? Results.NoContent()           // 204
        : Results.NotFound();           // 404
});
```

### Common `Results` Methods

| Method                       | Status Code | Use Case                          |
| ---------------------------- | ----------- | --------------------------------- |
| `Results.Ok(value)`          | 200         | Successful response with body     |
| `Results.Created(uri, val)`  | 201         | Resource created                  |
| `Results.Accepted(uri)`      | 202         | Accepted for processing           |
| `Results.NoContent()`        | 204         | Success with no body              |
| `Results.BadRequest(val)`    | 400         | Invalid request                   |
| `Results.Unauthorized()`     | 401         | Not authenticated                 |
| `Results.Forbid()`           | 403         | Not authorized                    |
| `Results.NotFound(val)`      | 404         | Resource not found                |
| `Results.Conflict(val)`      | 409         | Conflict with current state       |
| `Results.UnprocessableEntity(val)` | 422   | Validation failure                |
| `Results.Problem(detail)`    | 500         | Server error (ProblemDetails)     |
| `Results.ValidationProblem(errors)` | 400  | Validation errors (ProblemDetails)|
| `Results.File(bytes, ct)`    | 200         | File download                     |
| `Results.Stream(stream)`     | 200         | Stream response                   |
| `Results.Redirect(url)`      | 302         | Redirect                          |
| `Results.Json(val)`          | 200         | JSON with custom options          |
| `Results.Text(content)`      | 200         | Plain text                        |

### Returning ProblemDetails

ASP.NET Core uses **ProblemDetails** (RFC 7807) for standardized error responses. See [[Content Negotiation]] for how response formats are determined.

```csharp
app.MapGet("/products/{id}", async (int id, IProductService service) =>
{
    if (id <= 0)
    {
        return Results.Problem(
            detail: "Product ID must be a positive integer.",
            statusCode: 400,
            title: "Invalid Product ID");
    }

    var product = await service.GetByIdAsync(id);
    return product is not null
        ? Results.Ok(product)
        : Results.Problem(
            detail: $"No product found with ID {id}.",
            statusCode: 404,
            title: "Product Not Found");
});
```

### Returning Validation Errors

```csharp
app.MapPost("/products", async (CreateProductDto dto, IProductService service) =>
{
    var errors = new Dictionary<string, string[]>();

    if (string.IsNullOrWhiteSpace(dto.Name))
        errors["Name"] = new[] { "Product name is required." };

    if (dto.Price <= 0)
        errors["Price"] = new[] { "Price must be greater than zero." };

    if (errors.Count > 0)
        return Results.ValidationProblem(errors);

    var product = await service.CreateAsync(dto);
    return Results.Created($"/products/{product.Id}", product);
});
```

> [!summary] Section Summary
> Handlers can return strings, objects, or `IResult` values. The `Results` class provides factory methods for all standard HTTP responses including `Ok`, `NotFound`, `Created`, `BadRequest`, `ValidationProblem`, and `Problem`. Use `Results.ValidationProblem` for structured validation errors following RFC 7807.

---

## TypedResults vs Results

==.NET 7 introduced `TypedResults`==, which returns concrete types instead of `IResult`. This seemingly small difference has a major impact on OpenAPI documentation and testability.

### The Problem with `Results`

```csharp
// Returns IResult - OpenAPI doesn't know the response schema
app.MapGet("/products/{id}", async (int id, IProductService service) =>
{
    var product = await service.GetByIdAsync(id);
    return product is not null
        ? Results.Ok(product)       // Returns IResult
        : Results.NotFound();       // Returns IResult
});
```

With `Results`, the return type is `IResult` -- the framework cannot infer what types or status codes the endpoint produces, so OpenAPI documentation is incomplete.

### The Solution: `TypedResults`

```csharp
// Returns concrete types - OpenAPI knows exactly what to expect
app.MapGet("/products/{id}", async Task<Results<Ok<Product>, NotFound>> (
    int id, IProductService service) =>
{
    var product = await service.GetByIdAsync(id);
    return product is not null
        ? TypedResults.Ok(product)       // Returns Ok<Product>
        : TypedResults.NotFound();       // Returns NotFound
});
```

> [!warning]
> The `Results<T1, T2, ...>` union return type in the method signature is what enables OpenAPI inference. Without it, even `TypedResults` methods will not fully describe the endpoint. The union type can hold up to 6 type parameters: `Results<T1, T2, T3, T4, T5, T6>`.

### Side-by-Side Comparison

```csharp
// Using Results (IResult) -- no OpenAPI metadata inferred
app.MapGet("/v1/products/{id}", async (int id, IProductService svc) =>
{
    var product = await svc.GetByIdAsync(id);
    return product is not null
        ? Results.Ok(product)
        : Results.NotFound();
});

// Using TypedResults -- full OpenAPI metadata inferred
app.MapGet("/v2/products/{id}", async Task<Results<Ok<Product>, NotFound>> (
    int id, IProductService svc) =>
{
    var product = await svc.GetByIdAsync(id);
    return product is not null
        ? TypedResults.Ok(product)
        : TypedResults.NotFound();
});
```

### Testing Benefits

`TypedResults` makes unit testing cleaner because you get concrete types:

```csharp
[Fact]
public async Task GetProduct_ReturnsOk_WhenProductExists()
{
    // Arrange
    var service = new FakeProductService();
    service.Add(new Product(1, "Widget", 9.99m));

    // Act -- call the handler directly
    var result = await ProductEndpoints.GetById(1, service);

    // Assert -- result is a concrete type, not IResult
    var okResult = Assert.IsType<Ok<Product>>(result.Result);
    Assert.Equal("Widget", okResult.Value!.Name);
}
```

### When to Use Which

| Scenario                                  | Use              |
| ----------------------------------------- | ---------------- |
| Prototype / quick endpoint                | `Results`        |
| API with OpenAPI/Swagger documentation    | `TypedResults`   |
| Endpoints you want to unit test cleanly   | `TypedResults`   |
| Internal/private microservice API         | Either           |
| Public-facing API                         | `TypedResults`   |

> [!summary] Section Summary
> `TypedResults` returns concrete types (`Ok<T>`, `NotFound`, `Created<T>`) instead of `IResult`, enabling automatic OpenAPI metadata inference and cleaner unit tests. Use `Results<T1, T2, ...>` as the return type to express all possible responses. Prefer `TypedResults` for any API that needs documentation or testability.

---

## Route Groups

**Route groups** (.NET 7+) let you define a common prefix, filters, and metadata for a set of endpoints. They are created with `app.MapGroup()`.

### Basic Route Group

```csharp
var app = WebApplication.CreateBuilder(args).Build();

// Create a group with shared prefix
var products = app.MapGroup("/api/products");

products.MapGet("/", async (IProductService svc) =>
    Results.Ok(await svc.GetAllAsync()));                  // GET /api/products

products.MapGet("/{id}", async (int id, IProductService svc) =>
    Results.Ok(await svc.GetByIdAsync(id)));               // GET /api/products/{id}

products.MapPost("/", async (CreateProductDto dto, IProductService svc) =>
{
    var product = await svc.CreateAsync(dto);
    return Results.Created($"/api/products/{product.Id}", product);
});                                                         // POST /api/products

products.MapPut("/{id}", async (int id, UpdateProductDto dto, IProductService svc) =>
    Results.Ok(await svc.UpdateAsync(id, dto)));           // PUT /api/products/{id}

products.MapDelete("/{id}", async (int id, IProductService svc) =>
{
    await svc.DeleteAsync(id);
    return Results.NoContent();
});                                                         // DELETE /api/products/{id}

app.Run();
```

### Nested Groups

```csharp
var api = app.MapGroup("/api");

var products = api.MapGroup("/products");
products.MapGet("/", GetAllProducts);
products.MapGet("/{id}", GetProductById);

var orders = api.MapGroup("/orders");
orders.MapGet("/", GetAllOrders);
orders.MapGet("/{id}", GetOrderById);

// Results in: /api/products, /api/products/{id}, /api/orders, /api/orders/{id}
```

### Shared Metadata and Filters on Groups

```csharp
var admin = app.MapGroup("/api/admin")
    .RequireAuthorization("AdminPolicy")   // All endpoints require admin auth
    .AddEndpointFilter<ApiKeyFilter>()     // All endpoints go through API key filter
    .WithTags("Admin");                    // OpenAPI tag for all endpoints

admin.MapGet("/users", GetAllUsers);
admin.MapDelete("/users/{id}", DeleteUser);
admin.MapGet("/audit-log", GetAuditLog);
```

### Organizing Groups with Extension Methods

For clean code, define groups as extension methods:

```csharp
// In ProductEndpoints.cs
public static class ProductEndpoints
{
    public static RouteGroupBuilder MapProductEndpoints(this WebApplication app)
    {
        var group = app.MapGroup("/api/products")
            .WithTags("Products");

        group.MapGet("/", GetAll);
        group.MapGet("/{id}", GetById);
        group.MapPost("/", Create);
        group.MapPut("/{id}", Update);
        group.MapDelete("/{id}", Delete);

        return group;
    }

    private static async Task<IResult> GetAll(IProductService svc) =>
        Results.Ok(await svc.GetAllAsync());

    private static async Task<IResult> GetById(int id, IProductService svc) =>
        await svc.GetByIdAsync(id) is Product p
            ? Results.Ok(p)
            : Results.NotFound();

    private static async Task<IResult> Create(
        CreateProductDto dto, IProductService svc)
    {
        var product = await svc.CreateAsync(dto);
        return Results.Created($"/api/products/{product.Id}", product);
    }

    private static async Task<IResult> Update(
        int id, UpdateProductDto dto, IProductService svc) =>
        Results.Ok(await svc.UpdateAsync(id, dto));

    private static async Task<IResult> Delete(int id, IProductService svc)
    {
        await svc.DeleteAsync(id);
        return Results.NoContent();
    }
}

// In Program.cs -- one line per feature area
app.MapProductEndpoints();
app.MapOrderEndpoints();
app.MapCategoryEndpoints();
```

> [!tip]
> This extension method pattern is the recommended way to organize minimal APIs at scale. Each feature area gets its own file with a `Map{Feature}Endpoints()` method, keeping `Program.cs` focused on configuration.

> [!summary] Section Summary
> Route groups define a shared prefix, metadata, and filters for related endpoints. They support nesting and are best organized using extension methods on `WebApplication`. Each feature area should have its own `Map{Feature}Endpoints()` method for clean separation.

---

## Endpoint Filters

**Endpoint filters** (.NET 7+) are the minimal API equivalent of MVC action filters. They run before and after the endpoint handler, allowing you to add cross-cutting concerns like logging, validation, authorization, and transformation.

### Basic Endpoint Filter

```csharp
app.MapGet("/products/{id}", async (int id, IProductService svc) =>
    Results.Ok(await svc.GetByIdAsync(id)))
.AddEndpointFilter(async (context, next) =>
{
    var id = context.GetArgument<int>(0);
    if (id <= 0)
    {
        return Results.BadRequest("ID must be positive");
    }

    // Call the next filter or the endpoint handler
    return await next(context);
});
```

### Implementing `IEndpointFilter`

For reusable filters, implement the `IEndpointFilter` interface:

```csharp
public class ValidationFilter<T> : IEndpointFilter where T : class
{
    public async ValueTask<object?> InvokeAsync(
        EndpointFilterInvocationContext context,
        EndpointFilterDelegate next)
    {
        var argument = context.Arguments
            .OfType<T>()
            .FirstOrDefault();

        if (argument is null)
        {
            return Results.BadRequest("Request body is required.");
        }

        var validator = context.HttpContext
            .RequestServices
            .GetService<IValidator<T>>();

        if (validator is not null)
        {
            var validationResult = await validator.ValidateAsync(argument);
            if (!validationResult.IsValid)
            {
                return Results.ValidationProblem(
                    validationResult.ToDictionary());
            }
        }

        return await next(context);
    }
}

// Usage
app.MapPost("/products", async (CreateProductDto dto, IProductService svc) =>
{
    var product = await svc.CreateAsync(dto);
    return Results.Created($"/api/products/{product.Id}", product);
})
.AddEndpointFilter<ValidationFilter<CreateProductDto>>();
```

### Logging Filter

```csharp
public class LoggingFilter : IEndpointFilter
{
    private readonly ILogger<LoggingFilter> _logger;

    public LoggingFilter(ILogger<LoggingFilter> logger)
    {
        _logger = logger;
    }

    public async ValueTask<object?> InvokeAsync(
        EndpointFilterInvocationContext context,
        EndpointFilterDelegate next)
    {
        var path = context.HttpContext.Request.Path;
        var method = context.HttpContext.Request.Method;

        _logger.LogInformation("Handling {Method} {Path}", method, path);

        var stopwatch = Stopwatch.StartNew();
        var result = await next(context);
        stopwatch.Stop();

        _logger.LogInformation(
            "Handled {Method} {Path} in {Elapsed}ms",
            method, path, stopwatch.ElapsedMilliseconds);

        return result;
    }
}

// Apply to a group so all endpoints get logging
app.MapGroup("/api/products")
    .AddEndpointFilter<LoggingFilter>()
    .MapProductEndpoints();
```

### Filter Execution Order

Filters execute in the order they are added, forming a pipeline:

```csharp
app.MapPost("/products", CreateProduct)
    .AddEndpointFilter<LoggingFilter>()       // 1st: logs entry
    .AddEndpointFilter<AuthorizationFilter>() // 2nd: checks auth
    .AddEndpointFilter<ValidationFilter<CreateProductDto>>(); // 3rd: validates

// Execution order:
// LoggingFilter (before) -> AuthorizationFilter (before) -> ValidationFilter (before)
// -> Handler
// ValidationFilter (after) -> AuthorizationFilter (after) -> LoggingFilter (after)
```

> [!summary] Section Summary
> Endpoint filters are the minimal API equivalent of action filters. They execute before and after the handler, forming a pipeline. Implement `IEndpointFilter` for reusable filters. Filters can be applied to individual endpoints or entire route groups and execute in the order they are registered.

---

## Validation

Minimal APIs do ==not include built-in model validation== like the `[ApiController]` attribute provides for controllers. You must handle validation explicitly.

### Manual Validation

```csharp
app.MapPost("/products", async (CreateProductDto dto, IProductService svc) =>
{
    var errors = new Dictionary<string, string[]>();

    if (string.IsNullOrWhiteSpace(dto.Name))
        errors["Name"] = new[] { "Name is required." };
    else if (dto.Name.Length > 200)
        errors["Name"] = new[] { "Name cannot exceed 200 characters." };

    if (dto.Price <= 0)
        errors["Price"] = new[] { "Price must be greater than zero." };

    if (string.IsNullOrWhiteSpace(dto.Sku))
        errors["Sku"] = new[] { "SKU is required." };

    if (errors.Count > 0)
        return Results.ValidationProblem(errors);

    var product = await svc.CreateAsync(dto);
    return Results.Created($"/api/products/{product.Id}", product);
});
```

### Validation with FluentValidation

A more scalable approach uses **FluentValidation** with an endpoint filter:

```csharp
// Install: dotnet add package FluentValidation.DependencyInjectionExtensions
```

```csharp
// Validator definition
public class CreateProductValidator : AbstractValidator<CreateProductDto>
{
    public CreateProductValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Product name is required.")
            .MaximumLength(200).WithMessage("Name cannot exceed 200 characters.");

        RuleFor(x => x.Price)
            .GreaterThan(0).WithMessage("Price must be greater than zero.");

        RuleFor(x => x.Sku)
            .NotEmpty().WithMessage("SKU is required.")
            .Matches(@"^[A-Z]{2}-\d{4}$")
            .WithMessage("SKU must be in format XX-0000.");

        RuleFor(x => x.CategoryId)
            .GreaterThan(0).WithMessage("Valid category is required.");
    }
}
```

```csharp
// Generic validation filter using FluentValidation
public class FluentValidationFilter<T> : IEndpointFilter where T : class
{
    public async ValueTask<object?> InvokeAsync(
        EndpointFilterInvocationContext context,
        EndpointFilterDelegate next)
    {
        var dto = context.Arguments.OfType<T>().FirstOrDefault();

        if (dto is null)
            return Results.BadRequest("Request body is required.");

        var validator = context.HttpContext.RequestServices
            .GetService<IValidator<T>>();

        if (validator is not null)
        {
            var result = await validator.ValidateAsync(dto);
            if (!result.IsValid)
            {
                var errors = result.Errors
                    .GroupBy(e => e.PropertyName)
                    .ToDictionary(
                        g => g.Key,
                        g => g.Select(e => e.ErrorMessage).ToArray());

                return Results.ValidationProblem(errors);
            }
        }

        return await next(context);
    }
}
```

```csharp
// Registration
builder.Services.AddValidatorsFromAssemblyContaining<CreateProductValidator>();

// Usage
app.MapPost("/products", CreateProduct)
    .AddEndpointFilter<FluentValidationFilter<CreateProductDto>>();

app.MapPut("/products/{id}", UpdateProduct)
    .AddEndpointFilter<FluentValidationFilter<UpdateProductDto>>();
```

### Validation with Data Annotations and MiniValidator

```csharp
// Install: dotnet add package MiniValidation
using MiniValidation;

public record CreateProductDto(
    [Required, StringLength(200)] string Name,
    [Range(0.01, 999999.99)] decimal Price,
    [Required, RegularExpression(@"^[A-Z]{2}-\d{4}$")] string Sku);

app.MapPost("/products", async (CreateProductDto dto, IProductService svc) =>
{
    if (!MiniValidator.TryValidate(dto, out var errors))
        return Results.ValidationProblem(errors);

    var product = await svc.CreateAsync(dto);
    return Results.Created($"/api/products/{product.Id}", product);
});
```

> [!warning]
> Unlike controllers with `[ApiController]`, minimal APIs do not automatically validate Data Annotation attributes. You must invoke validation explicitly, either manually, via a library like MiniValidation/FluentValidation, or through an endpoint filter.

> [!summary] Section Summary
> Minimal APIs require explicit validation since there is no automatic model validation. Options include manual validation, FluentValidation with endpoint filters (recommended for complex APIs), or MiniValidation for Data Annotations. Always return `Results.ValidationProblem()` for structured error responses.

---

## OpenAPI and Swagger Integration

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

---

## File Uploads

Minimal APIs support file uploads through `IFormFile` and `IFormFileCollection` parameter binding.

### Single File Upload

```csharp
app.MapPost("/products/{id}/image", async (
    int id,
    IFormFile file,
    IProductService svc) =>
{
    if (file.Length == 0)
        return Results.BadRequest("File is empty.");

    if (file.Length > 5 * 1024 * 1024) // 5 MB limit
        return Results.BadRequest("File exceeds 5 MB limit.");

    var allowedTypes = new[] { "image/jpeg", "image/png", "image/webp" };
    if (!allowedTypes.Contains(file.ContentType))
        return Results.BadRequest("Only JPEG, PNG, and WebP images are allowed.");

    using var stream = file.OpenReadStream();
    var imageUrl = await svc.UploadImageAsync(id, stream, file.FileName);

    return Results.Ok(new { imageUrl });
})
.DisableAntiforgery()  // Required for API file uploads
.Accepts<IFormFile>("multipart/form-data")
.WithName("UploadProductImage");
```

### Multiple File Upload

```csharp
app.MapPost("/products/{id}/images", async (
    int id,
    IFormFileCollection files,
    IProductService svc) =>
{
    if (files.Count == 0)
        return Results.BadRequest("No files provided.");

    if (files.Count > 5)
        return Results.BadRequest("Maximum 5 files allowed.");

    var urls = new List<string>();
    foreach (var file in files)
    {
        using var stream = file.OpenReadStream();
        var url = await svc.UploadImageAsync(id, stream, file.FileName);
        urls.Add(url);
    }

    return Results.Ok(new { urls });
})
.DisableAntiforgery();
```

### File Upload with Additional Form Fields (.NET 8+)

```csharp
app.MapPost("/products", async (
    [FromForm] string name,
    [FromForm] decimal price,
    [FromForm] IFormFile image,
    IProductService svc) =>
{
    using var stream = image.OpenReadStream();
    var product = await svc.CreateWithImageAsync(name, price, stream, image.FileName);
    return Results.Created($"/api/products/{product.Id}", product);
})
.DisableAntiforgery();
```

Calling the endpoint:

```bash
curl -X POST https://localhost:5001/products \
  -F "name=Widget" \
  -F "price=9.99" \
  -F "image=@photo.jpg"
```

> [!warning]
> Always call `.DisableAntiforgery()` on file upload endpoints intended for API consumers. Without it, the endpoint expects an antiforgery token, which API clients do not provide.

> [!summary] Section Summary
> File uploads use `IFormFile` (single) or `IFormFileCollection` (multiple) as handler parameters. Always validate file size and content type. Call `.DisableAntiforgery()` for API-facing upload endpoints. .NET 8+ supports mixed form fields and file uploads using `[FromForm]`.

---

## Minimal APIs vs Controllers

Choosing between minimal APIs and [[API Controllers]] depends on the project's size, team, and requirements.

### Feature Comparison

| Feature                       | Minimal APIs                      | Controllers                          |
| ----------------------------- | --------------------------------- | ------------------------------------ |
| Startup code                  | Very little                       | More boilerplate                     |
| File organization             | Flexible (any file)               | Convention-driven (Controllers/)     |
| Routing                       | Fluent API (`MapGet`, etc.)       | Attribute or conventional routing    |
| Model validation              | Manual / endpoint filters         | Automatic with `[ApiController]`     |
| Action filters                | Endpoint filters (.NET 7+)        | Full filter pipeline (5 types)       |
| Model binding                 | Automatic + attributes            | Automatic + attributes               |
| Content negotiation           | Supported                         | Full support with formatters         |
| API versioning                | Via packages/groups               | Built-in conventions                 |
| OpenAPI                       | Metadata methods / TypedResults   | Attribute-based / conventions        |
| Dependency injection          | Handler parameters                | Constructor injection                |
| Testability                   | Good (TypedResults)               | Good (ActionResult<T>)              |
| `IActionFilter` / `IAsyncActionFilter` | Not available             | Full support                         |
| `OutputFormatter` negotiation | Limited                           | Full pipeline                        |
| OData support                 | Not supported                     | Supported                            |

### When to Use Minimal APIs

- **Microservices** with a small number of focused endpoints
- **Prototyping** and proof-of-concept applications
- **Serverless functions** (Azure Functions-style handlers)
- **Small APIs** (under ~20-30 endpoints)
- **BFF (Backend-for-Frontend)** layers that proxy to other services
- When you want **maximum control** with minimum ceremony
- **Learning ASP.NET Core** -- lower barrier to entry

### When to Use Controllers

- **Large APIs** with dozens or hundreds of endpoints
- When you need the **full filter pipeline** (authorization, resource, action, result, exception filters)
- When you rely on **OData** for queryable endpoints
- When your team is **familiar with MVC conventions**
- When you need **complex content negotiation** with multiple output formatters
- **Enterprise applications** where convention-over-configuration is preferred

### Hybrid Approach

You can use both in the same application:

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers(); // Add controller support

var app = builder.Build();

// Minimal API endpoints
app.MapGet("/health", () => Results.Ok("Healthy"));
app.MapProductEndpoints(); // Minimal API group

// Controller endpoints
app.MapControllers(); // Map attribute-routed controllers

app.Run();
```

> [!example]
> A common pattern: use minimal APIs for simple health checks, configuration endpoints, and lightweight microservice routes, while using controllers for the main business API that benefits from filters and conventions.

> [!summary] Section Summary
> Minimal APIs excel at small, focused APIs, microservices, prototyping, and serverless scenarios. Controllers are better for large APIs needing the full filter pipeline, OData, or complex content negotiation. Both can coexist in the same application. Choose based on the project's complexity and team familiarity.

---

## Real-World Example: Complete CRUD API

This section brings everything together into a production-style minimal API for managing products, with DI, validation, Swagger, and proper error handling.

### Project Structure

```
ProductApi/
  Program.cs
  Models/
    Product.cs
    CreateProductDto.cs
    UpdateProductDto.cs
    ProductResponse.cs
  Services/
    IProductService.cs
    ProductService.cs
  Validators/
    CreateProductValidator.cs
    UpdateProductValidator.cs
  Endpoints/
    ProductEndpoints.cs
  Data/
    AppDbContext.cs
```

### Models

```csharp
// Models/Product.cs
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Sku { get; set; } = string.Empty;
    public string? Description { get; set; }
    public decimal Price { get; set; }
    public int CategoryId { get; set; }
    public bool IsActive { get; set; } = true;
    public DateTime CreatedAt { get; set; }
    public DateTime? UpdatedAt { get; set; }
}

// Models/CreateProductDto.cs
public record CreateProductDto(
    string Name,
    string Sku,
    string? Description,
    decimal Price,
    int CategoryId);

// Models/UpdateProductDto.cs
public record UpdateProductDto(
    string Name,
    string? Description,
    decimal Price,
    int CategoryId,
    bool IsActive);

// Models/ProductResponse.cs
public record ProductResponse(
    int Id,
    string Name,
    string Sku,
    string? Description,
    decimal Price,
    int CategoryId,
    bool IsActive,
    DateTime CreatedAt);
```

### DbContext

```csharp
// Data/AppDbContext.cs
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Product> Products => Set<Product>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Product>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Name).HasMaxLength(200).IsRequired();
            entity.Property(e => e.Sku).HasMaxLength(20).IsRequired();
            entity.HasIndex(e => e.Sku).IsUnique();
            entity.Property(e => e.Price).HasPrecision(18, 2);
            entity.Property(e => e.Description).HasMaxLength(2000);
        });
    }
}
```

### Service Layer

```csharp
// Services/IProductService.cs
public interface IProductService
{
    Task<IEnumerable<ProductResponse>> GetAllAsync(
        string? name, int page, int pageSize);
    Task<ProductResponse?> GetByIdAsync(int id);
    Task<ProductResponse> CreateAsync(CreateProductDto dto);
    Task<ProductResponse?> UpdateAsync(int id, UpdateProductDto dto);
    Task<bool> DeleteAsync(int id);
    Task<bool> SkuExistsAsync(string sku, int? excludeId = null);
}

// Services/ProductService.cs
public class ProductService : IProductService
{
    private readonly AppDbContext _db;

    public ProductService(AppDbContext db) => _db = db;

    public async Task<IEnumerable<ProductResponse>> GetAllAsync(
        string? name, int page, int pageSize)
    {
        var query = _db.Products.AsQueryable();

        if (!string.IsNullOrWhiteSpace(name))
            query = query.Where(p => p.Name.Contains(name));

        return await query
            .OrderBy(p => p.Name)
            .Skip((page - 1) * pageSize)
            .Take(pageSize)
            .Select(p => ToResponse(p))
            .ToListAsync();
    }

    public async Task<ProductResponse?> GetByIdAsync(int id)
    {
        var product = await _db.Products.FindAsync(id);
        return product is null ? null : ToResponse(product);
    }

    public async Task<ProductResponse> CreateAsync(CreateProductDto dto)
    {
        var product = new Product
        {
            Name = dto.Name,
            Sku = dto.Sku,
            Description = dto.Description,
            Price = dto.Price,
            CategoryId = dto.CategoryId,
            CreatedAt = DateTime.UtcNow
        };

        _db.Products.Add(product);
        await _db.SaveChangesAsync();

        return ToResponse(product);
    }

    public async Task<ProductResponse?> UpdateAsync(int id, UpdateProductDto dto)
    {
        var product = await _db.Products.FindAsync(id);
        if (product is null) return null;

        product.Name = dto.Name;
        product.Description = dto.Description;
        product.Price = dto.Price;
        product.CategoryId = dto.CategoryId;
        product.IsActive = dto.IsActive;
        product.UpdatedAt = DateTime.UtcNow;

        await _db.SaveChangesAsync();
        return ToResponse(product);
    }

    public async Task<bool> DeleteAsync(int id)
    {
        var product = await _db.Products.FindAsync(id);
        if (product is null) return false;

        _db.Products.Remove(product);
        await _db.SaveChangesAsync();
        return true;
    }

    public async Task<bool> SkuExistsAsync(string sku, int? excludeId = null)
    {
        return await _db.Products
            .AnyAsync(p => p.Sku == sku && (!excludeId.HasValue || p.Id != excludeId));
    }

    private static ProductResponse ToResponse(Product p) =>
        new(p.Id, p.Name, p.Sku, p.Description, p.Price,
            p.CategoryId, p.IsActive, p.CreatedAt);
}
```

### Validators (FluentValidation)

```csharp
// Validators/CreateProductValidator.cs
public class CreateProductValidator : AbstractValidator<CreateProductDto>
{
    private readonly IProductService _service;

    public CreateProductValidator(IProductService service)
    {
        _service = service;

        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Product name is required.")
            .MaximumLength(200).WithMessage("Name cannot exceed 200 characters.");

        RuleFor(x => x.Sku)
            .NotEmpty().WithMessage("SKU is required.")
            .Matches(@"^[A-Z]{2}-\d{4}$").WithMessage("SKU format: XX-0000")
            .MustAsync(async (sku, ct) => !await _service.SkuExistsAsync(sku))
            .WithMessage("SKU already exists.");

        RuleFor(x => x.Price)
            .GreaterThan(0).WithMessage("Price must be positive.")
            .LessThanOrEqualTo(999999.99m).WithMessage("Price cannot exceed 999,999.99.");

        RuleFor(x => x.CategoryId)
            .GreaterThan(0).WithMessage("Valid category required.");
    }
}

// Validators/UpdateProductValidator.cs
public class UpdateProductValidator : AbstractValidator<UpdateProductDto>
{
    public UpdateProductValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Product name is required.")
            .MaximumLength(200).WithMessage("Name cannot exceed 200 characters.");

        RuleFor(x => x.Price)
            .GreaterThan(0).WithMessage("Price must be positive.");

        RuleFor(x => x.CategoryId)
            .GreaterThan(0).WithMessage("Valid category required.");
    }
}
```

### Reusable Validation Filter

```csharp
// Filters/FluentValidationFilter.cs
public class FluentValidationFilter<T> : IEndpointFilter where T : class
{
    public async ValueTask<object?> InvokeAsync(
        EndpointFilterInvocationContext context,
        EndpointFilterDelegate next)
    {
        var dto = context.Arguments.OfType<T>().FirstOrDefault();

        if (dto is null)
            return Results.BadRequest(new { error = "Request body is required." });

        var validator = context.HttpContext.RequestServices
            .GetService<IValidator<T>>();

        if (validator is not null)
        {
            var result = await validator.ValidateAsync(dto);
            if (!result.IsValid)
            {
                return Results.ValidationProblem(
                    result.Errors
                        .GroupBy(e => e.PropertyName)
                        .ToDictionary(
                            g => g.Key,
                            g => g.Select(e => e.ErrorMessage).ToArray()));
            }
        }

        return await next(context);
    }
}
```

### Endpoint Registration

```csharp
// Endpoints/ProductEndpoints.cs
public static class ProductEndpoints
{
    public static void MapProductEndpoints(this WebApplication app)
    {
        var group = app.MapGroup("/api/products")
            .WithTags("Products")
            .WithOpenApi();

        group.MapGet("/", GetAll)
            .WithName("GetProducts")
            .WithSummary("Search products with pagination")
            .Produces<IEnumerable<ProductResponse>>(200);

        group.MapGet("/{id:int}", GetById)
            .WithName("GetProductById")
            .WithSummary("Get a product by ID")
            .Produces<ProductResponse>(200)
            .Produces(404);

        group.MapPost("/", Create)
            .WithName("CreateProduct")
            .WithSummary("Create a new product")
            .Produces<ProductResponse>(201)
            .ProducesValidationProblem()
            .AddEndpointFilter<FluentValidationFilter<CreateProductDto>>();

        group.MapPut("/{id:int}", Update)
            .WithName("UpdateProduct")
            .WithSummary("Update an existing product")
            .Produces<ProductResponse>(200)
            .Produces(404)
            .ProducesValidationProblem()
            .AddEndpointFilter<FluentValidationFilter<UpdateProductDto>>();

        group.MapDelete("/{id:int}", Delete)
            .WithName("DeleteProduct")
            .WithSummary("Delete a product")
            .Produces(204)
            .Produces(404);
    }

    private static async Task<IResult> GetAll(
        [FromQuery] string? name,
        [FromQuery] int? page,
        [FromQuery] int? pageSize,
        IProductService service)
    {
        var products = await service.GetAllAsync(
            name,
            page ?? 1,
            Math.Clamp(pageSize ?? 20, 1, 100));

        return Results.Ok(products);
    }

    private static async Task<IResult> GetById(
        int id, IProductService service)
    {
        var product = await service.GetByIdAsync(id);
        return product is not null
            ? Results.Ok(product)
            : Results.NotFound(new { error = $"Product {id} not found." });
    }

    private static async Task<IResult> Create(
        CreateProductDto dto, IProductService service)
    {
        var product = await service.CreateAsync(dto);
        return Results.Created($"/api/products/{product.Id}", product);
    }

    private static async Task<IResult> Update(
        int id, UpdateProductDto dto, IProductService service)
    {
        var product = await service.UpdateAsync(id, dto);
        return product is not null
            ? Results.Ok(product)
            : Results.NotFound(new { error = $"Product {id} not found." });
    }

    private static async Task<IResult> Delete(
        int id, IProductService service)
    {
        var deleted = await service.DeleteAsync(id);
        return deleted
            ? Results.NoContent()
            : Results.NotFound(new { error = $"Product {id} not found." });
    }
}
```

### Program.cs

```csharp
using FluentValidation;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Database
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")));

// Services
builder.Services.AddScoped<IProductService, ProductService>();

// Validation
builder.Services.AddValidatorsFromAssemblyContaining<CreateProductValidator>();

// OpenAPI / Swagger
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new Microsoft.OpenApi.Models.OpenApiInfo
    {
        Title = "Product API",
        Version = "v1",
        Description = "A minimal API for managing products"
    });
});

var app = builder.Build();

// Middleware
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

// Map endpoints
app.MapProductEndpoints();

// Health check
app.MapGet("/health", () => Results.Ok(new
{
    status = "Healthy",
    timestamp = DateTime.UtcNow
}))
.WithTags("System")
.ExcludeFromDescription();

app.Run();
```

### Testing the API

```http
### Get all products
GET https://localhost:5001/api/products?page=1&pageSize=10

### Get product by ID
GET https://localhost:5001/api/products/1

### Create a product
POST https://localhost:5001/api/products
Content-Type: application/json

{
    "name": "Wireless Mouse",
    "sku": "WM-1001",
    "description": "Ergonomic wireless mouse with Bluetooth",
    "price": 29.99,
    "categoryId": 3
}

### Update a product
PUT https://localhost:5001/api/products/1
Content-Type: application/json

{
    "name": "Wireless Mouse Pro",
    "description": "Updated ergonomic wireless mouse",
    "price": 39.99,
    "categoryId": 3,
    "isActive": true
}

### Delete a product
DELETE https://localhost:5001/api/products/1
```

```json
// Example validation error response (RFC 7807)
{
    "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
    "title": "One or more validation errors occurred.",
    "status": 400,
    "errors": {
        "Name": ["Product name is required."],
        "Sku": ["SKU format: XX-0000"],
        "Price": ["Price must be positive."]
    }
}
```

> [!summary] Section Summary
> A production-grade minimal API uses route groups for organization, FluentValidation with endpoint filters for validation, a service layer for business logic, DTOs for request/response shaping, and Swagger for documentation. Handlers are organized into static classes with extension methods, keeping `Program.cs` clean.

---

## Comprehensive Summary

> [!summary] Comprehensive Summary
>
> **Minimal APIs** are a lightweight approach to building HTTP APIs in ASP.NET Core, introduced in .NET 6, where endpoints are defined directly using `Map{Verb}` methods on `WebApplication`.
>
> **Core Concepts:**
> - Endpoints are defined with lambda expressions or method groups mapped to HTTP verbs and route patterns
> - Parameters are automatically bound from routes, query strings, request bodies, headers, and the DI container
> - The `Results` class provides factory methods for all standard HTTP responses
> - `TypedResults` (.NET 7+) enables automatic OpenAPI metadata inference and cleaner testing
>
> **Organization and Scaling:**
> - Route groups share prefixes, filters, and metadata across related endpoints
> - Extension methods like `Map{Feature}Endpoints()` keep `Program.cs` clean
> - Endpoint filters provide the cross-cutting concern pipeline (logging, validation, auth)
>
> **Key Differences from Controllers:**
> - No automatic model validation -- must use manual checks, FluentValidation, or MiniValidation
> - No `IActionFilter` -- use endpoint filters instead
> - No OData support
> - Better for microservices, prototyping, and small APIs
> - Controllers remain better for large, convention-heavy enterprise APIs
>
> **Best Practices:**
> - Use `TypedResults` with `Results<T1, T2, ...>` return types for OpenAPI and testability
> - Organize endpoints into feature-specific static classes with `MapGroup`
> - Use FluentValidation with a generic `IEndpointFilter` for validation
> - Apply shared concerns (auth, logging, tags) at the group level
> - Both minimal APIs and controllers can coexist in the same application
>
> **See Also:** [[API Controllers]] | [[Content Negotiation]] | [[API Conventions]]
