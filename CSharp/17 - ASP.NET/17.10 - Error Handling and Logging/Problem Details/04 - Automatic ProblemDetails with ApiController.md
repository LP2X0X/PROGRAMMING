---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - problem-details
  - api
---


The `[ApiController]` attribute transforms how controllers handle errors. Understanding what it does automatically prevents confusion when debugging unexpected response formats.

## Automatic Model Validation

Without `[ApiController]`, you must check `ModelState` manually:

```csharp
// Without [ApiController] -- manual validation
[HttpPost]
public IActionResult CreateProduct(CreateProductRequest request)
{
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);  // Returns a different format than ProblemDetails
    }
    // ...
}
```

With `[ApiController]`, invalid model state is handled before your action executes:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpPost]
    public IActionResult CreateProduct(CreateProductRequest request)
    {
        // If request fails validation, this code NEVER RUNS.
        // The framework returns ValidationProblemDetails automatically.

        var product = _service.Create(request);
        return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
    }
}
```

## Customizing the Automatic Validation Response

You can customize the automatic validation response by configuring `ApiBehaviorOptions`:

```csharp
builder.Services.AddControllers()
    .ConfigureApiBehaviorOptions(options =>
    {
        options.InvalidModelStateResponseFactory = context =>
        {
            var problemDetails = new ValidationProblemDetails(context.ModelState)
            {
                Type = "https://example.com/errors/validation",
                Title = "Validation Failed",
                Status = StatusCodes.Status422UnprocessableEntity,
                Detail = "See the errors property for details.",
                Instance = context.HttpContext.Request.Path
            };

            problemDetails.Extensions["traceId"] = 
                context.HttpContext.TraceIdentifier;

            return new UnprocessableEntityObjectResult(problemDetails)
            {
                ContentTypes = { "application/problem+json" }
            };
        };
    });
```

> [!warning] Common Misconception
> Many developers are surprised when their action method never executes for invalid input. With `[ApiController]`, model validation happens in a **model validation action filter** that runs before the action. If `ModelState` is invalid, the filter short-circuits and returns the validation response immediately. You do not need (and should not add) `if (!ModelState.IsValid)` checks in `[ApiController]` controllers -- it is redundant.

> [!summary] Section Summary
> - `[ApiController]` intercepts invalid model state before the action method runs, returning `ValidationProblemDetails` automatically
> - This eliminates the need for manual `ModelState.IsValid` checks in API controllers
> - Customize the automatic response format via `ApiBehaviorOptions.InvalidModelStateResponseFactory`
> - The default returns 400; you can change this to 422 (Unprocessable Entity) if you prefer RFC 4918 semantics
