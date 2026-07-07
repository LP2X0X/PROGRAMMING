---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - problem-details
  - api
---


For validation errors (typically 400 Bad Request or 422 Unprocessable Entity), ASP.NET Core provides **`ValidationProblemDetails`**, which extends `ProblemDetails` with an `errors` dictionary mapping field names to their validation messages.

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "detail": null,
  "instance": "/api/products",
  "errors": {
    "Name": [
      "The Name field is required.",
      "The Name field must be between 3 and 100 characters."
    ],
    "Price": [
      "The Price field must be greater than 0."
    ]
  },
  "traceId": "00-abc123..."
}
```

## The ValidationProblemDetails Class

```csharp
// Inherits from ProblemDetails
public class ValidationProblemDetails : ProblemDetails
{
    // Maps property names to arrays of validation error messages
    public IDictionary<string, string[]> Errors { get; set; }
}
```

## Creating ValidationProblemDetails Manually

```csharp
[HttpPost]
public IActionResult CreateProduct(CreateProductRequest request)
{
    var errors = new Dictionary<string, string[]>();

    if (string.IsNullOrWhiteSpace(request.Name))
        errors["Name"] = new[] { "Name is required." };

    if (request.Price <= 0)
        errors["Price"] = new[] { "Price must be greater than zero." };

    if (errors.Count > 0)
    {
        return ValidationProblem(new ValidationProblemDetails(errors)
        {
            Detail = "Please correct the validation errors and try again.",
            Instance = HttpContext.Request.Path
        });
    }

    // ... create the product ...
    return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
}
```

> [!tip]
> With `[ApiController]`, you rarely need to create `ValidationProblemDetails` manually. The framework automatically returns a 400 response with `ValidationProblemDetails` when model validation fails (i.e., when `ModelState.IsValid` is false). The automatic behavior kicks in *before* your action method even executes.

> [!summary] Section Summary
> - `ValidationProblemDetails` extends `ProblemDetails` with an `errors` dictionary mapping field names to validation messages
> - `[ApiController]` automatically returns `ValidationProblemDetails` for model binding/validation failures
> - The format is consistent and machine-parseable -- client libraries can generically display field-level errors
> - Create manually when you have custom validation logic beyond data annotations
