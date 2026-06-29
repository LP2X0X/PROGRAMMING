---
tags:
  - csharp
  - asp-net-core
  - minimal-apis
  - web-api
---


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
