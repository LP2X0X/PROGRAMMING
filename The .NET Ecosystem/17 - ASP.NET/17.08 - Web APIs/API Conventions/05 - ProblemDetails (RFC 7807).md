---
tags:
  - csharp
  - asp-net-core
  - web-api
  - conventions
  - openapi
---


**ProblemDetails** is a ==standardized machine-readable format for describing errors in HTTP APIs==, defined in RFC 7807 (updated by RFC 9457). ASP.NET Core has built-in support for generating ProblemDetails responses, ensuring that every error your API returns follows a consistent, predictable structure.

### The ProblemDetails Structure

A ProblemDetails response is a JSON object with these properties:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.5",
  "title": "Not Found",
  "status": 404,
  "detail": "Product with ID 42 was not found.",
  "instance": "/api/products/42",
  "traceId": "00-abc123def456-789ghi-00"
}
```

| Property | Description |
|---|---|
| `type` | A URI reference identifying the problem type. Defaults to the RFC section for the status code |
| `title` | A short, human-readable summary of the problem |
| `status` | The HTTP status code |
| `detail` | A human-readable explanation specific to this occurrence |
| `instance` | A URI reference identifying the specific occurrence (usually the request path) |
| `extensions` | Additional properties (like `traceId`) added by middleware or custom code |

### Enabling ProblemDetails Globally

In .NET 7+, you can enable ProblemDetails for all error responses with a single call:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddProblemDetails(); // Enable ProblemDetails globally

var app = builder.Build();

app.UseExceptionHandler();  // Converts unhandled exceptions to ProblemDetails
app.UseStatusCodePages();   // Converts empty error responses to ProblemDetails

app.MapControllers();
app.Run();
```

With this configuration:
- **Unhandled exceptions** return a 500 ProblemDetails response (without leaking stack traces in production)
- **Empty error responses** (like returning `NotFound()` with no body) are enriched with ProblemDetails
- **Model validation failures** already return ProblemDetails by default when using `[ApiController]`

### Validation Errors as ProblemDetails

When `[ApiController]` is applied, model validation failures automatically produce a **ValidationProblemDetails** response:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "Name": ["The Name field is required."],
    "Price": ["The Price must be between 0.01 and 99999.99."]
  },
  "traceId": "00-abc123..."
}
```

### Customizing ProblemDetails Responses

#### Using AddProblemDetails with a Configure Action

```csharp
builder.Services.AddProblemDetails(options =>
{
    options.CustomizeProblemDetails = context =>
    {
        // Add the trace ID to every ProblemDetails response
        context.ProblemDetails.Instance =
            $"{context.HttpContext.Request.Method} " +
            $"{context.HttpContext.Request.Path}";

        context.ProblemDetails.Extensions["requestId"] =
            context.HttpContext.TraceIdentifier;

        context.ProblemDetails.Extensions["nodeId"] =
            Environment.MachineName;
    };
});
```

#### Creating Custom Problem Types

Define domain-specific error types for richer error responses:

```csharp
[HttpPost]
public async Task<IActionResult> CreateOrder(CreateOrderDto dto)
{
    if (!await _inventoryService.HasSufficientStock(dto.ProductId, dto.Quantity))
    {
        return Problem(
            type: "https://myapi.com/errors/insufficient-stock",
            title: "Insufficient Stock",
            detail: $"Product {dto.ProductId} only has " +
                    $"{await _inventoryService.GetStock(dto.ProductId)} units " +
                    $"available, but {dto.Quantity} were requested.",
            statusCode: StatusCodes.Status409Conflict
        );
    }

    var order = await _orderService.CreateAsync(dto);
    return CreatedAtAction(nameof(GetOrder), new { id = order.Id }, order);
}
```

This produces:

```json
{
  "type": "https://myapi.com/errors/insufficient-stock",
  "title": "Insufficient Stock",
  "status": 409,
  "detail": "Product 7 only has 3 units available, but 10 were requested."
}
```

#### Custom Exception Handler with ProblemDetails

```csharp
app.UseExceptionHandler(exceptionApp =>
{
    exceptionApp.Run(async context =>
    {
        var exceptionFeature = context.Features
            .Get<IExceptionHandlerPathFeature>();
        var exception = exceptionFeature?.Error;

        var problemDetails = exception switch
        {
            EntityNotFoundException e => new ProblemDetails
            {
                Type = "https://myapi.com/errors/not-found",
                Title = "Resource Not Found",
                Status = StatusCodes.Status404NotFound,
                Detail = e.Message,
                Instance = context.Request.Path
            },
            BusinessRuleViolationException e => new ProblemDetails
            {
                Type = "https://myapi.com/errors/business-rule-violation",
                Title = "Business Rule Violation",
                Status = StatusCodes.Status422UnprocessableEntity,
                Detail = e.Message,
                Instance = context.Request.Path
            },
            _ => new ProblemDetails
            {
                Type = "https://tools.ietf.org/html/rfc9110#section-15.6.1",
                Title = "Internal Server Error",
                Status = StatusCodes.Status500InternalServerError,
                Detail = "An unexpected error occurred.",
                Instance = context.Request.Path
            }
        };

        context.Response.StatusCode = problemDetails.Status
                                      ?? StatusCodes.Status500InternalServerError;
        await context.Response.WriteAsJsonAsync(problemDetails);
    });
});
```

### ProblemDetails in Minimal APIs

ProblemDetails works the same way in [[Minimal APIs]]:

```csharp
app.MapGet("/api/products/{id}", async (int id, ProductRepository repo) =>
{
    var product = await repo.GetByIdAsync(id);
    return product is not null
        ? Results.Ok(product)
        : Results.Problem(
            title: "Product Not Found",
            detail: $"No product exists with ID {id}.",
            statusCode: StatusCodes.Status404NotFound);
});
```

> [!tip]
> In .NET 8+, `Results.Problem()` and `TypedResults.Problem()` support the full ProblemDetails parameter set. Use `TypedResults` for better OpenAPI metadata inference in [[Minimal APIs]].

> [!summary] Section Summary
> ProblemDetails (RFC 7807) provides a standardized JSON error format with `type`, `title`, `status`, `detail`, and `instance` fields. Enable globally with `AddProblemDetails()`, `UseExceptionHandler()`, and `UseStatusCodePages()`. Customize with domain-specific problem types and custom exception handlers.
