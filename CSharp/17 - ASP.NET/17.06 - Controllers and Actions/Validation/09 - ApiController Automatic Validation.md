---
tags:
  - csharp
  - asp-net-core
  - validation
  - model-binding
---


When you decorate a controller with `[ApiController]`, the framework adds several API-specific behaviors, one of which is **automatic model validation**. You do **not** need to check `ModelState.IsValid` yourself.

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(CreateProductRequest request)
    {
        // No ModelState.IsValid check needed!
        // If validation fails, the framework returns 400 before this code runs.
        var product = _productService.Create(request);
        return CreatedAtAction(nameof(Get), new { id = product.Id }, product);
    }
}
```

### How It Works

The `[ApiController]` attribute registers a built-in **action filter** called `ModelStateInvalidFilter`. This filter runs before your action method and:

1. Checks if `ModelState.IsValid` is `false`
2. If invalid, short-circuits the pipeline and returns a **400 Bad Request** response
3. The response body is a `ValidationProblemDetails` object (RFC 7807 Problem Details format)

### The Automatic 400 Response

The automatic response follows the Problem Details standard:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "Name": ["The Name field is required."],
    "Price": ["The field Price must be between 0.01 and 99999.99."],
    "ContactEmail": ["The ContactEmail field is not a valid e-mail address."]
  },
  "traceId": "00-abc123def456-789ghi-00"
}
```

```ad-info
The `ValidationProblemDetails` response includes a `traceId` that correlates with your server-side logging, making it easier to debug validation issues in production.
```

### Disabling Automatic Validation

If you need to handle invalid model state yourself (for example, to add business-rule errors before returning), disable the automatic filter:

```csharp
builder.Services.AddControllers()
    .ConfigureApiBehaviorOptions(options =>
    {
        options.SuppressModelStateInvalidFilter = true;
    });
```

You can also customize the response factory instead of disabling it entirely:

```csharp
builder.Services.AddControllers()
    .ConfigureApiBehaviorOptions(options =>
    {
        options.InvalidModelStateResponseFactory = context =>
        {
            var problemDetails = new ValidationProblemDetails(context.ModelState)
            {
                Title = "Validation failed",
                Status = StatusCodes.Status422UnprocessableEntity,
                Instance = context.HttpContext.Request.Path
            };

            return new UnprocessableEntityObjectResult(problemDetails);
        };
    });
```

```ad-summary
`[ApiController]` adds automatic model validation via `ModelStateInvalidFilter`. Invalid requests get a 400 response with `ValidationProblemDetails` (RFC 7807) before your action code runs. Disable with `SuppressModelStateInvalidFilter = true`, or customize the response with `InvalidModelStateResponseFactory`.
```
