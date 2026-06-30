---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - problem-details
  - api
---


ASP.NET Core provides two main mechanisms for automatic ProblemDetails generation: the `[ApiController]` attribute and the `AddProblemDetails()` service configuration.

## ApiController Automatic Behavior

The `[ApiController]` attribute on a controller class enables several automatic behaviors, including ProblemDetails responses:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetProduct(int id)
    {
        var product = _repository.GetById(id);

        if (product is null)
            return NotFound();  // Returns ProblemDetails JSON with status 404

        return Ok(product);
    }

    [HttpPost]
    public IActionResult CreateProduct(CreateProductRequest request)
    {
        // If model validation fails, ApiController automatically returns
        // ValidationProblemDetails with 400 status BEFORE this code runs
        // ...
    }
}
```

When `[ApiController]` is present:

| Scenario | Behavior |
|---|---|
| Model validation fails | Returns `ValidationProblemDetails` (400) automatically before the action executes |
| Action returns `NotFound()` | Returns ProblemDetails with status 404 |
| Action returns `BadRequest()` | Returns ProblemDetails with status 400 |
| Action returns `Conflict()` | Returns ProblemDetails with status 409 |
| Action returns any error `ObjectResult` | Wraps it in ProblemDetails format |

## AddProblemDetails Service

.NET 7+ introduced `AddProblemDetails()` to enable ProblemDetails across the **entire application**, not just `[ApiController]` controllers:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddProblemDetails();

var app = builder.Build();

// Now exception handler and status code pages also produce ProblemDetails
app.UseExceptionHandler();
app.UseStatusCodePages();

app.MapGet("/api/test", () =>
{
    throw new InvalidOperationException("Something broke");
});

app.Run();
```

With `AddProblemDetails()` registered, the following middleware automatically generates ProblemDetails responses:
- `UseExceptionHandler()` -- unhandled exceptions return ProblemDetails with 500
- `UseStatusCodePages()` -- non-exception error status codes return ProblemDetails
- Developer Exception Page -- includes ProblemDetails when the client sends `Accept: application/json`

> [!summary] Section Summary
> - `[ApiController]` automatically returns ProblemDetails for error status codes and validation failures
> - `AddProblemDetails()` (.NET 7+) extends ProblemDetails to the entire pipeline, including exception handlers and status code pages
> - Together, they ensure consistent JSON error responses across all API endpoints
> - Validation failures return `ValidationProblemDetails` with the `errors` dictionary automatically
