---
tags:
  - csharp
  - asp-net-core
  - action-results
  - controllers
---


ProblemDetails is a standardized format (RFC 7807 / RFC 9457) for returning machine-readable error information in HTTP API responses. ASP.NET Core has built-in support for it.

### What a ProblemDetails Response Looks Like

```json
{
    "type": "https://tools.ietf.org/html/rfc9110#section-15.5.5",
    "title": "Not Found",
    "status": 404,
    "detail": "Product with ID 42 was not found.",
    "traceId": "00-abc123def456-789ghi-00"
}
```

For validation errors, the framework uses `ValidationProblemDetails`, which adds an `errors` dictionary:

```json
{
    "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
    "title": "One or more validation errors occurred.",
    "status": 400,
    "errors": {
        "Name": ["The Name field is required."],
        "Price": ["The Price must be greater than 0."]
    },
    "traceId": "00-abc123def456-789ghi-00"
}
```

### Automatic ProblemDetails with \[ApiController\]

When you apply `[ApiController]` to a controller, the framework automatically:

1. Converts `ModelState` validation failures into `ValidationProblemDetails` (400)
2. Wraps client error status codes (4xx) in `ProblemDetails` format

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet("{id}")]
    public ActionResult<Product> GetProduct(int id)
    {
        var product = _repository.Find(id);

        if (product is null)
            return NotFound();   // automatically formatted as ProblemDetails JSON

        return product;
    }
}
```

### Enabling ProblemDetails Globally

In `Program.cs`, you can configure ProblemDetails for the entire application, including for exceptions:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

// Enable ProblemDetails for all error responses, including unhandled exceptions
builder.Services.AddProblemDetails();

var app = builder.Build();

// The exception handler and status code pages use ProblemDetails format
app.UseExceptionHandler();
app.UseStatusCodePages();

app.MapControllers();
app.Run();
```

### Customizing ProblemDetails

You can customize the ProblemDetails output by configuring `ProblemDetailsOptions`:

```csharp
builder.Services.AddProblemDetails(options =>
{
    options.CustomizeProblemDetails = context =>
    {
        context.ProblemDetails.Instance =
            $"{context.HttpContext.Request.Method} {context.HttpContext.Request.Path}";

        context.ProblemDetails.Extensions["requestId"] =
            context.HttpContext.TraceIdentifier;

        context.ProblemDetails.Extensions["serverTime"] =
            DateTime.UtcNow.ToString("O");
    };
});
```

### Returning ProblemDetails Manually

You can also return `ProblemDetails` explicitly using the `Problem()` helper:

```csharp
[HttpGet("{id}")]
public ActionResult<Product> GetProduct(int id)
{
    var product = _repository.Find(id);

    if (product is null)
    {
        return Problem(
            detail: $"Product with ID {id} was not found.",
            statusCode: StatusCodes.Status404NotFound,
            title: "Product Not Found"
        );
    }

    return product;
}
```

```ad-attention
By default, in `Development` environment, ProblemDetails may include the exception stack trace for 500 errors. In production, it omits sensitive details. Make sure you test both environments to verify what clients actually see.
```
