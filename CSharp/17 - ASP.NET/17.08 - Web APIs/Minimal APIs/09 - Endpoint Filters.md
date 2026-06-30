---
tags:
  - csharp
  - asp-net-core
  - minimal-apis
  - web-api
---


**Endpoint filters** (.NET 7+) are the minimal API equivalent of MVC action filters. They run before and after the endpoint handler, allowing you to add cross-cutting concerns like logging, validation, authorization, and transformation.

### Similarities with Middleware

Endpoint filters and [[Middleware Overview|middleware]] share the same core pattern:

- Both use a ==before/after== model around a `next()` call — code above `next()` runs before, code below runs after
- Both can ==short-circuit== by returning a response without calling `next()`, skipping everything downstream
- Both are suited for ==cross-cutting concerns== like logging, validation, and error handling

### Key Differences from Middleware

Despite the similarities, endpoint filters and middleware operate at different levels:

| | Middleware | Endpoint Filters |
|---|---|---|
| **Scope** | Runs for ==all requests== (static files, non-endpoint requests, everything) | Runs ==only for requests that reach the endpoint== |
| **Access to endpoint details** | Sees only the final HTTP response (status code, headers, body) | Has access to the ==endpoint's return value== (e.g. the `IResult` object) before it becomes an HTTP response |
| **Targeting** | Applies globally to the entire pipeline by default | Can easily target a ==single endpoint or route group== |

> [!ad-tip] When to Use Which
> - Use **middleware** for concerns that apply to ==every request==: logging, CORS, exception handling, HTTPS redirection
> - Use **endpoint filters** for concerns specific to ==your API endpoints==: input validation, request/response transformation, per-endpoint authorization checks

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
