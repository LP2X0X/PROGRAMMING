---
tags: [csharp, asp-net-core, error-handling, problem-details, api]
aliases: [ProblemDetails, RFC 9457, RFC 7807, API Error Responses]
status: complete
date: 2026-06-18
---

# Problem Details

When an API returns an error, what should the response body look like? Without a standard, every API invents its own format -- `{ "error": "..." }`, `{ "message": "...", "code": 123 }`, `{ "errors": [...] }` -- and every client must be taught each API's specific error shape. This is the problem that **ProblemDetails** solves.

**ProblemDetails** is a standardized error response format defined by **RFC 9457** (formerly RFC 7807), titled "Problem Details for HTTP APIs." It provides a machine-readable, extensible JSON structure for communicating errors from HTTP APIs. ASP.NET Core has built-in support for generating ProblemDetails responses, making it straightforward to adopt this standard across all your API endpoints.

---

## Table of Contents

- [[#The ProblemDetails Standard -- RFC 9457]]
- [[#The ProblemDetails Shape]]
- [[#ValidationProblemDetails -- Validation Errors]]
- [[#Built-in ProblemDetails Support in ASP.NET Core]]
- [[#Automatic ProblemDetails with ApiController]]
- [[#Configuring ProblemDetails Globally]]
- [[#Customizing ProblemDetails Responses]]
- [[#IProblemDetailsService]]
- [[#ProblemDetails in Minimal APIs]]
- [[#Adding Custom Extension Members]]
- [[#ProblemDetails for Different Audiences]]
- [[#Integration with IExceptionHandler]]
- [[#Real-World -- Production API with ProblemDetails]]
- [[#Related Topics]]
- [[#Further Reading]]
- [[#Comprehensive Summary]]

---

## The ProblemDetails Standard -- RFC 9457

RFC 9457 defines a standard way for HTTP APIs to communicate errors to clients. The key motivation is **consistency**: if every API uses the same error format, client libraries can handle errors generically instead of needing custom parsing logic per API.

The standard is:
- **Media type**: `application/problem+json` (or `application/problem+xml`)
- **Not tied to any framework** -- it is a pure HTTP/JSON standard used by APIs written in any language
- **Extensible** -- you can add custom members beyond the standard fields
- **Machine-readable** -- the `type` field is a URI that identifies the error type programmatically

### History

- **RFC 7807** (2016): Original specification by M. Nottingham and E. Wilde
- **RFC 9457** (2023): Supersedes RFC 7807 with clarifications and minor updates
- ASP.NET Core adopted ProblemDetails support starting in ASP.NET Core 2.1, with significant improvements in .NET 7 and .NET 8

> [!info] Definition
> **ProblemDetails** = a standardized JSON (or XML) format for conveying machine-readable details of errors in an HTTP response, as defined by RFC 9457. It provides a consistent shape that any HTTP client can parse, regardless of which API generated the error.

> [!summary] Section Summary
> - ProblemDetails (RFC 9457, formerly RFC 7807) is the industry standard for HTTP API error responses
> - It provides a consistent, machine-readable JSON format with well-defined fields
> - The media type `application/problem+json` signals that the response body is a ProblemDetails object
> - Adopted by ASP.NET Core since version 2.1, with full support in .NET 7/8

---

## The ProblemDetails Shape

The standard defines five core members, all optional:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.5",
  "title": "Not Found",
  "status": 404,
  "detail": "Product with ID 42 was not found.",
  "instance": "/api/products/42"
}
```

### Standard Members

| Member | Type | Purpose | Example |
|---|---|---|---|
| **`type`** | string (URI) | A URI reference that identifies the **problem type**. Ideally points to human-readable documentation. | `"https://example.com/errors/product-not-found"` |
| **`title`** | string | A short, human-readable summary of the problem type. Should be the same for every occurrence of this problem type. | `"Not Found"` |
| **`status`** | integer | The HTTP status code. Included for convenience -- it must match the actual HTTP response status code. | `404` |
| **`detail`** | string | A human-readable explanation specific to **this occurrence** of the problem. Unlike `title`, this changes per request. | `"Product with ID 42 was not found."` |
| **`instance`** | string (URI) | A URI reference that identifies the **specific occurrence** of the problem. Often the request path or a unique error ID. | `"/api/products/42"` |

### ASP.NET Core Additions

ASP.NET Core's `ProblemDetails` class adds a commonly used member:

| Member | Type | Purpose |
|---|---|---|
| **`extensions`** | dictionary | A bag for custom key-value pairs (e.g., `traceId`, `errorCode`, `timestamp`) |

In practice, ASP.NET Core automatically includes a `traceId` extension:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.5",
  "title": "Not Found",
  "status": 404,
  "detail": "Product with ID 42 was not found.",
  "instance": "/api/products/42",
  "traceId": "00-84f1c85e03d87d4cb7eafab94ddf2f58-7f3e26f21c3b254a-00"
}
```

### The ProblemDetails Class in C#

```csharp
// Microsoft.AspNetCore.Mvc.ProblemDetails
public class ProblemDetails
{
    public string? Type { get; set; }
    public string? Title { get; set; }
    public int? Status { get; set; }
    public string? Detail { get; set; }
    public string? Instance { get; set; }

    // Extension members -- any additional key-value pairs
    [JsonExtensionData]
    public IDictionary<string, object?> Extensions { get; set; }
}
```

> [!warning] Common Misconception
> The `type` field is not just a descriptive string -- it is meant to be a **URI** that a client can use to programmatically identify the error type. While the RFC does not require that the URI be dereferenceable (i.e., it does not need to return a web page), it is best practice to point it to documentation that describes the error. Many teams use `about:blank` or an HTTP status reference URI as a default when they have not set up custom error type documentation.

> [!ad-note]
> All members are optional per the RFC. However, you should always include at least `type`, `title`, and `status` for a useful error response. The `detail` field should describe **this specific error occurrence**, not the error type in general.

> [!summary] Section Summary
> - ProblemDetails has five standard members: `type` (URI identifying the problem), `title` (human-readable summary), `status` (HTTP code), `detail` (instance-specific explanation), `instance` (request URI or unique error reference)
> - ASP.NET Core adds an `Extensions` dictionary for custom data like `traceId`
> - The `type` field should be a URI -- ideally pointing to documentation -- not just a descriptive string
> - `title` is the same for every occurrence of a problem type; `detail` is specific to each occurrence

---

## ValidationProblemDetails -- Validation Errors

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

### The ValidationProblemDetails Class

```csharp
// Inherits from ProblemDetails
public class ValidationProblemDetails : ProblemDetails
{
    // Maps property names to arrays of validation error messages
    public IDictionary<string, string[]> Errors { get; set; }
}
```

### Creating ValidationProblemDetails Manually

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

---

## Built-in ProblemDetails Support in ASP.NET Core

ASP.NET Core provides two main mechanisms for automatic ProblemDetails generation: the `[ApiController]` attribute and the `AddProblemDetails()` service configuration.

### ApiController Automatic Behavior

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

### AddProblemDetails Service

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

---

## Automatic ProblemDetails with ApiController

The `[ApiController]` attribute transforms how controllers handle errors. Understanding what it does automatically prevents confusion when debugging unexpected response formats.

### Automatic Model Validation

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

### Customizing the Automatic Validation Response

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

---

## Configuring ProblemDetails Globally

`AddProblemDetails()` accepts a configuration delegate that lets you customize every ProblemDetails response generated by the framework:

```csharp
builder.Services.AddProblemDetails(options =>
{
    options.CustomizeProblemDetails = context =>
    {
        // Add traceId to every ProblemDetails response
        context.ProblemDetails.Extensions["traceId"] = 
            context.HttpContext.TraceIdentifier;

        // Add the machine name for debugging in multi-node deployments
        context.ProblemDetails.Extensions["nodeId"] = 
            Environment.MachineName;

        // Add a timestamp
        context.ProblemDetails.Extensions["timestamp"] = 
            DateTime.UtcNow.ToString("o");

        // Set the instance to the request path if not already set
        context.ProblemDetails.Instance ??= 
            context.HttpContext.Request.Path;

        // Set a default type URI based on status code
        if (string.IsNullOrEmpty(context.ProblemDetails.Type))
        {
            var statusCode = context.ProblemDetails.Status ?? 500;
            context.ProblemDetails.Type = 
                $"https://httpstatuses.com/{statusCode}";
        }
    };
});
```

This customization applies to ProblemDetails generated by:
- `UseExceptionHandler()` middleware
- `UseStatusCodePages()` middleware
- `[ApiController]` automatic responses
- Manual calls to `Problem()`, `ValidationProblem()`, or `Results.Problem()`

> [!ad-note]
> The `CustomizeProblemDetails` delegate runs for **every** ProblemDetails response, regardless of how it was generated. This is the single best place to add cross-cutting concerns like trace IDs, timestamps, and node identifiers.

> [!summary] Section Summary
> - `AddProblemDetails(options => ...)` customizes all ProblemDetails responses globally
> - Use `CustomizeProblemDetails` to add trace IDs, timestamps, node identifiers, and default `type` URIs
> - The customization applies to all ProblemDetails from any source: exception handlers, status code pages, `[ApiController]`, and manual creation
> - This is the best place for cross-cutting metadata that should appear on every error response

---

## Customizing ProblemDetails Responses

Beyond global configuration, you can customize ProblemDetails at the point of creation for specific error scenarios.

### In Controllers

```csharp
[HttpGet("{id}")]
public IActionResult GetProduct(int id)
{
    var product = _repository.GetById(id);

    if (product is null)
    {
        return Problem(
            type: "https://example.com/errors/product-not-found",
            title: "Product Not Found",
            statusCode: StatusCodes.Status404NotFound,
            detail: $"No product exists with ID {id}.",
            instance: HttpContext.Request.Path
        );
    }

    return Ok(product);
}
```

### Creating ProblemDetails Objects Directly

```csharp
[HttpDelete("{id}")]
public IActionResult DeleteProduct(int id)
{
    var product = _repository.GetById(id);

    if (product is null)
        return NotFound();

    if (product.HasActiveOrders)
    {
        var problemDetails = new ProblemDetails
        {
            Type = "https://example.com/errors/product-has-orders",
            Title = "Cannot Delete Product",
            Status = StatusCodes.Status409Conflict,
            Detail = $"Product '{product.Name}' has {product.ActiveOrderCount} " +
                     "active orders and cannot be deleted.",
            Instance = HttpContext.Request.Path
        };

        problemDetails.Extensions["productId"] = id;
        problemDetails.Extensions["activeOrderCount"] = product.ActiveOrderCount;

        return new ObjectResult(problemDetails)
        {
            StatusCode = StatusCodes.Status409Conflict,
            ContentTypes = { "application/problem+json" }
        };
    }

    _repository.Delete(id);
    return NoContent();
}
```

> [!summary] Section Summary
> - Use the `Problem()` helper method on `ControllerBase` for quick ProblemDetails responses
> - Create `ProblemDetails` objects directly when you need to add extension members
> - Set `ContentTypes` to `"application/problem+json"` on the `ObjectResult` for correct media type
> - Include relevant context in extension members (IDs, counts, timestamps) that help clients handle the error

---

## IProblemDetailsService

The **`IProblemDetailsService`** interface (.NET 7+) lets you generate ProblemDetails programmatically from anywhere in your application -- not just from controllers. It is registered automatically when you call `AddProblemDetails()`.

```csharp
public interface IProblemDetailsService
{
    ValueTask WriteAsync(ProblemDetailsContext context);
}
```

### Using IProblemDetailsService in Middleware

```csharp
public class RateLimitingMiddleware
{
    private readonly RequestDelegate _next;

    public RateLimitingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(
        HttpContext context,
        IProblemDetailsService problemDetailsService)
    {
        if (IsRateLimited(context))
        {
            context.Response.StatusCode = StatusCodes.Status429TooManyRequests;

            await problemDetailsService.WriteAsync(new ProblemDetailsContext
            {
                HttpContext = context,
                ProblemDetails =
                {
                    Type = "https://example.com/errors/rate-limited",
                    Title = "Too Many Requests",
                    Status = 429,
                    Detail = "You have exceeded the rate limit. " +
                             "Please wait before making another request.",
                }
            });

            return;
        }

        await _next(context);
    }

    private static bool IsRateLimited(HttpContext context)
    {
        // Rate limiting logic...
        return false;
    }
}
```

### Using IProblemDetailsService in Minimal APIs

```csharp
app.MapGet("/api/products/{id}", async (
    int id,
    IProductRepository repository,
    IProblemDetailsService problemDetailsService,
    HttpContext context) =>
{
    var product = await repository.GetByIdAsync(id);

    if (product is null)
    {
        context.Response.StatusCode = 404;
        await problemDetailsService.WriteAsync(new ProblemDetailsContext
        {
            HttpContext = context,
            ProblemDetails =
            {
                Type = "https://example.com/errors/product-not-found",
                Title = "Product Not Found",
                Detail = $"No product exists with ID {id}."
            }
        });
        return;
    }

    await context.Response.WriteAsJsonAsync(product);
});
```

> [!tip]
> `IProblemDetailsService` respects the global `CustomizeProblemDetails` configuration. When you write ProblemDetails through this service, your global customizations (trace ID, node ID, etc.) are automatically applied. This ensures consistency regardless of where the error is generated.

> [!summary] Section Summary
> - `IProblemDetailsService` generates ProblemDetails from any component -- middleware, minimal APIs, services
> - Registered automatically by `AddProblemDetails()`
> - Respects global `CustomizeProblemDetails` configuration, ensuring consistency across all error sources
> - Inject it where you need to produce standardized error responses outside of controller actions

---

## ProblemDetails in Minimal APIs

Minimal APIs have their own helpers for generating ProblemDetails responses:

```csharp
// Results.Problem() -- returns ProblemDetails
app.MapGet("/api/products/{id}", (int id) =>
{
    var product = repository.GetById(id);

    return product is not null
        ? Results.Ok(product)
        : Results.Problem(
            type: "https://example.com/errors/product-not-found",
            title: "Product Not Found",
            statusCode: StatusCodes.Status404NotFound,
            detail: $"No product exists with ID {id}.",
            instance: $"/api/products/{id}");
});

// TypedResults.Problem() -- for typed return values
app.MapGet("/api/products/{id}", Results<Ok<Product>, ProblemHttpResult> (int id) =>
{
    var product = repository.GetById(id);

    return product is not null
        ? TypedResults.Ok(product)
        : TypedResults.Problem(
            title: "Product Not Found",
            statusCode: StatusCodes.Status404NotFound,
            detail: $"No product exists with ID {id}.");
});

// Results.ValidationProblem() -- for validation errors
app.MapPost("/api/products", (CreateProductRequest request) =>
{
    var errors = Validate(request);

    if (errors.Count > 0)
    {
        return Results.ValidationProblem(errors,
            title: "Validation Failed",
            detail: "One or more fields have invalid values.");
    }

    var product = repository.Create(request);
    return Results.Created($"/api/products/{product.Id}", product);
});
```

### ProblemDetails with Filters in Minimal APIs

```csharp
// Global endpoint filter that catches domain exceptions
app.MapGroup("/api")
    .AddEndpointFilter(async (context, next) =>
    {
        try
        {
            return await next(context);
        }
        catch (NotFoundException ex)
        {
            return Results.Problem(
                title: "Not Found",
                statusCode: 404,
                detail: ex.Message);
        }
        catch (ConflictException ex)
        {
            return Results.Problem(
                title: "Conflict",
                statusCode: 409,
                detail: ex.Message);
        }
    });
```

> [!summary] Section Summary
> - `Results.Problem()` and `TypedResults.Problem()` generate ProblemDetails in minimal APIs
> - `Results.ValidationProblem()` generates `ValidationProblemDetails` with the `errors` dictionary
> - Use `TypedResults` for OpenAPI documentation support (the return types are visible to the schema generator)
> - Endpoint filters can catch domain exceptions and convert them to ProblemDetails for a group of endpoints

---

## Adding Custom Extension Members

The ProblemDetails standard allows custom extension members beyond the five standard fields. These appear as additional JSON properties at the top level of the response.

```json
{
  "type": "https://example.com/errors/insufficient-funds",
  "title": "Insufficient Funds",
  "status": 422,
  "detail": "Account balance is $50.00 but the transaction requires $75.00.",
  "instance": "/api/transactions/789",
  "traceId": "00-abc123...",
  "errorCode": "INSUFFICIENT_FUNDS",
  "currentBalance": 50.00,
  "requiredAmount": 75.00,
  "supportUrl": "https://support.example.com/billing"
}
```

### Adding Extensions in Code

```csharp
var problemDetails = new ProblemDetails
{
    Type = "https://example.com/errors/insufficient-funds",
    Title = "Insufficient Funds",
    Status = 422,
    Detail = $"Account balance is {balance:C} but the transaction requires {amount:C}."
};

// Use the Extensions dictionary
problemDetails.Extensions["errorCode"] = "INSUFFICIENT_FUNDS";
problemDetails.Extensions["currentBalance"] = balance;
problemDetails.Extensions["requiredAmount"] = amount;
problemDetails.Extensions["supportUrl"] = "https://support.example.com/billing";

return new ObjectResult(problemDetails) { StatusCode = 422 };
```

### Common Extension Members

| Extension | Purpose | Example Value |
|---|---|---|
| `traceId` | Correlate with server logs | `"00-84f1c85e..."` |
| `errorCode` | Machine-readable error identifier | `"PRODUCT_NOT_FOUND"` |
| `timestamp` | When the error occurred | `"2026-06-18T14:30:00Z"` |
| `helpUrl` | Link to documentation about the error | `"https://docs.example.com/errors/auth"` |
| `retryAfter` | When to retry (for rate limiting) | `60` (seconds) |
| `validationErrors` | Detailed validation context | `[{ "field": "email", "rule": "format" }]` |

> [!tip]
> Choose **stable, documented** extension member names and keep them consistent across your entire API. Clients will build logic around these names. Treat extension names as part of your API contract -- changing them is a breaking change. Document your custom extensions in your API reference alongside the standard ProblemDetails members.

> [!ad-note]
> Extensions are serialized as top-level properties in the JSON response due to the `[JsonExtensionData]` attribute on the `Extensions` dictionary. They do not appear inside a nested `"extensions"` object -- they sit alongside `type`, `title`, `status`, etc.

> [!summary] Section Summary
> - ProblemDetails supports custom extension members via the `Extensions` dictionary
> - Extensions appear as top-level JSON properties alongside the standard members
> - Common extensions: `errorCode`, `traceId`, `timestamp`, `helpUrl`, `retryAfter`
> - Treat extension names as part of your API contract -- keep them stable and documented

---

## ProblemDetails for Different Audiences

A well-designed ProblemDetails response serves two audiences simultaneously:

### Machine Clients (Programs, SDKs)

Machine clients use:
- **`type`** -- to identify the error programmatically and branch logic accordingly
- **`status`** -- to determine the HTTP status code without parsing headers
- **`errors`** (ValidationProblemDetails) -- to map field-level errors to form fields
- **Custom extensions** (e.g., `errorCode`) -- for specific business logic branching

```csharp
// Client-side code that handles ProblemDetails
var response = await httpClient.PostAsJsonAsync("/api/orders", order);

if (!response.IsSuccessStatusCode)
{
    var problem = await response.Content
        .ReadFromJsonAsync<ProblemDetails>();

    switch (problem?.Type)
    {
        case "https://example.com/errors/insufficient-funds":
            var balance = problem.Extensions["currentBalance"];
            ShowInsufficientFundsDialog(balance);
            break;

        case "https://example.com/errors/product-unavailable":
            ShowProductUnavailableMessage();
            break;

        default:
            ShowGenericError(problem?.Detail ?? "Unknown error");
            break;
    }
}
```

### Human Users (Developers, Support Staff)

Human readers use:
- **`title`** -- a quick summary of what went wrong
- **`detail`** -- the specific explanation for this occurrence
- **`instance`** -- which request caused the error
- **`traceId`** -- to find the full details in server logs

### Design Guidelines

```csharp
// GOOD: Serves both audiences
new ProblemDetails
{
    // Machine-readable identifier
    Type = "https://example.com/errors/order-limit-exceeded",

    // Same for every occurrence of this problem type
    Title = "Order Limit Exceeded",

    Status = 422,

    // Specific to this occurrence -- human-readable
    Detail = "Customer 'Acme Corp' has reached the monthly order limit " +
             "of 1000 orders. Current count: 1000. " +
             "Limit resets on 2026-07-01.",

    // The request that caused it
    Instance = "/api/orders",

    Extensions =
    {
        // Machine-readable extension for programmatic handling
        ["errorCode"] = "ORDER_LIMIT_EXCEEDED",
        ["currentCount"] = 1000,
        ["limit"] = 1000,
        ["resetsAt"] = "2026-07-01T00:00:00Z",
        ["traceId"] = httpContext.TraceIdentifier
    }
};
```

> [!warning] Common Misconception
> Do not conflate `title` and `detail`. The `title` is a **generic description of the problem type** -- it should be the same for every occurrence (like "Not Found" or "Validation Failed"). The `detail` is **specific to this occurrence** -- it should describe exactly what went wrong with this particular request. Putting instance-specific information in `title` defeats the purpose of having both fields.

> [!summary] Section Summary
> - ProblemDetails serves two audiences: machines (via `type`, `status`, extensions) and humans (via `title`, `detail`, `instance`)
> - `type` is the primary machine-readable identifier for branching client logic
> - `title` is generic to the problem type; `detail` is specific to the occurrence
> - Client SDKs can switch on `type` URI to handle different error scenarios programmatically
> - Include enough context in extensions for clients to react without needing additional API calls

---

## Integration with IExceptionHandler

The most powerful pattern combines `IExceptionHandler` (.NET 8+) with ProblemDetails to convert domain exceptions into standardized API error responses. This is where [[Exception Handling]] and ProblemDetails come together.

```csharp
public class DomainExceptionHandler : IExceptionHandler
{
    private readonly IProblemDetailsService _problemDetailsService;
    private readonly ILogger<DomainExceptionHandler> _logger;

    public DomainExceptionHandler(
        IProblemDetailsService problemDetailsService,
        ILogger<DomainExceptionHandler> logger)
    {
        _problemDetailsService = problemDetailsService;
        _logger = logger;
    }

    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        // Map the exception to a ProblemDetails response
        var (statusCode, type, title, errorCode) = exception switch
        {
            NotFoundException => (
                404,
                "https://example.com/errors/not-found",
                "Resource Not Found",
                "NOT_FOUND"),

            ConflictException => (
                409,
                "https://example.com/errors/conflict",
                "Resource Conflict",
                "CONFLICT"),

            ValidationException => (
                422,
                "https://example.com/errors/validation",
                "Validation Failed",
                "VALIDATION_FAILED"),

            ForbiddenException => (
                403,
                "https://example.com/errors/forbidden",
                "Access Denied",
                "ACCESS_DENIED"),

            _ => (0, "", "", "")  // Not handled by this handler
        };

        if (statusCode == 0)
            return false;  // Pass to the next handler

        _logger.LogWarning(exception,
            "Domain exception {ErrorCode} on {Method} {Path}",
            errorCode,
            httpContext.Request.Method,
            httpContext.Request.Path);

        httpContext.Response.StatusCode = statusCode;

        var problemDetails = new ProblemDetails
        {
            Type = type,
            Title = title,
            Status = statusCode,
            Detail = exception.Message,
            Instance = httpContext.Request.Path
        };

        problemDetails.Extensions["errorCode"] = errorCode;

        // Handle ValidationException specially -- include field errors
        if (exception is ValidationException validationEx)
        {
            problemDetails = new ValidationProblemDetails(validationEx.Errors)
            {
                Type = type,
                Title = title,
                Status = statusCode,
                Detail = exception.Message,
                Instance = httpContext.Request.Path
            };
            problemDetails.Extensions["errorCode"] = errorCode;
        }

        // Use IProblemDetailsService to apply global customizations
        await _problemDetailsService.WriteAsync(new ProblemDetailsContext
        {
            HttpContext = httpContext,
            ProblemDetails = problemDetails
        });

        return true;
    }
}
```

### Registration

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddProblemDetails(options =>
{
    options.CustomizeProblemDetails = ctx =>
    {
        ctx.ProblemDetails.Extensions["traceId"] = 
            ctx.HttpContext.TraceIdentifier;
    };
});

builder.Services.AddExceptionHandler<DomainExceptionHandler>();
builder.Services.AddExceptionHandler<FallbackExceptionHandler>();

var app = builder.Build();

app.UseExceptionHandler();
app.MapControllers();
app.Run();
```

> [!tip]
> Using `IProblemDetailsService` inside your `IExceptionHandler` (instead of writing JSON directly) ensures that the global `CustomizeProblemDetails` customizations are applied. If you use `WriteAsJsonAsync` directly, you bypass the global configuration and lose consistency.

> [!summary] Section Summary
> - `IExceptionHandler` combined with `IProblemDetailsService` provides the cleanest integration of [[Exception Handling]] and ProblemDetails
> - Map domain exceptions to specific `type` URIs, `title` strings, and `errorCode` extensions via pattern matching
> - Use `IProblemDetailsService.WriteAsync()` instead of direct JSON serialization to ensure global customizations are applied
> - Handle `ValidationException` specially by using `ValidationProblemDetails` with field-level errors
> - Register specific handlers before the catch-all to maintain clean separation of concerns

---

## Real-World -- Production API with ProblemDetails

Here is a complete, production-ready API setup that uses ProblemDetails for all error responses with custom error codes and full traceability.

### Program.cs

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register ProblemDetails with global customizations
builder.Services.AddProblemDetails(options =>
{
    options.CustomizeProblemDetails = ctx =>
    {
        ctx.ProblemDetails.Extensions["traceId"] = 
            ctx.HttpContext.TraceIdentifier;
        ctx.ProblemDetails.Extensions["timestamp"] = 
            DateTimeOffset.UtcNow;

        // Set a default type URI if none was provided
        if (string.IsNullOrEmpty(ctx.ProblemDetails.Type))
        {
            var statusCode = ctx.ProblemDetails.Status 
                ?? ctx.HttpContext.Response.StatusCode;
            ctx.ProblemDetails.Type = 
                $"https://httpstatuses.com/{statusCode}";
        }
    };
});

// Register exception handlers
builder.Services.AddExceptionHandler<DomainExceptionHandler>();
builder.Services.AddExceptionHandler<FallbackExceptionHandler>();

builder.Services.AddControllers()
    .ConfigureApiBehaviorOptions(options =>
    {
        // Customize the automatic validation response
        options.InvalidModelStateResponseFactory = context =>
        {
            var problemDetails = new ValidationProblemDetails(context.ModelState)
            {
                Type = "https://example.com/errors/validation",
                Title = "Validation Failed",
                Status = StatusCodes.Status422UnprocessableEntity,
                Instance = context.HttpContext.Request.Path
            };

            problemDetails.Extensions["errorCode"] = "VALIDATION_FAILED";
            problemDetails.Extensions["traceId"] = 
                context.HttpContext.TraceIdentifier;
            problemDetails.Extensions["timestamp"] = DateTimeOffset.UtcNow;

            return new ObjectResult(problemDetails)
            {
                StatusCode = StatusCodes.Status422UnprocessableEntity,
                ContentTypes = { "application/problem+json" }
            };
        };
    });

var app = builder.Build();

app.UseExceptionHandler();
app.UseStatusCodePages();
app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### FallbackExceptionHandler.cs

```csharp
public class FallbackExceptionHandler : IExceptionHandler
{
    private readonly IProblemDetailsService _problemDetailsService;
    private readonly ILogger<FallbackExceptionHandler> _logger;
    private readonly IHostEnvironment _environment;

    public FallbackExceptionHandler(
        IProblemDetailsService problemDetailsService,
        ILogger<FallbackExceptionHandler> logger,
        IHostEnvironment environment)
    {
        _problemDetailsService = problemDetailsService;
        _logger = logger;
        _environment = environment;
    }

    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        _logger.LogError(exception,
            "Unhandled exception on {Method} {Path}",
            httpContext.Request.Method,
            httpContext.Request.Path);

        httpContext.Response.StatusCode = 500;

        await _problemDetailsService.WriteAsync(new ProblemDetailsContext
        {
            HttpContext = httpContext,
            ProblemDetails =
            {
                Type = "https://httpstatuses.com/500",
                Title = "Internal Server Error",
                Status = 500,
                Detail = _environment.IsDevelopment()
                    ? exception.Message
                    : "An unexpected error occurred. Please try again later.",
            },
            Exception = exception
        });

        return true;
    }
}
```

### Example API Controller

```csharp
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _service;
    private readonly ILogger<ProductsController> _logger;

    public ProductsController(
        IProductService service,
        ILogger<ProductsController> logger)
    {
        _service = service;
        _logger = logger;
    }

    [HttpGet("{id}")]
    [ProducesResponseType(typeof(Product), StatusCodes.Status200OK)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status404NotFound)]
    public async Task<IActionResult> GetProduct(int id)
    {
        // If the product does not exist, the service throws NotFoundException
        // which is caught by DomainExceptionHandler and converted to ProblemDetails
        var product = await _service.GetByIdAsync(id);
        return Ok(product);
    }

    [HttpPost]
    [ProducesResponseType(typeof(Product), StatusCodes.Status201Created)]
    [ProducesResponseType(typeof(ValidationProblemDetails),
        StatusCodes.Status422UnprocessableEntity)]
    public async Task<IActionResult> CreateProduct(CreateProductRequest request)
    {
        // Model validation is handled automatically by [ApiController]
        // If the service detects a business rule violation, it throws a 
        // DomainException which becomes ProblemDetails via the handler
        var product = await _service.CreateAsync(request);

        _logger.LogInformation("Product {ProductId} created: {Name}",
            product.Id, product.Name);

        return CreatedAtAction(
            nameof(GetProduct),
            new { id = product.Id },
            product);
    }

    [HttpDelete("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status404NotFound)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status409Conflict)]
    public async Task<IActionResult> DeleteProduct(int id)
    {
        // Throws NotFoundException if product does not exist
        // Throws ConflictException if product has active orders
        await _service.DeleteAsync(id);

        _logger.LogInformation("Product {ProductId} deleted", id);

        return NoContent();
    }
}
```

### What the Client Sees

Successful response:

```json
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 42,
  "name": "Widget Pro",
  "price": 29.99
}
```

Not found:

```json
HTTP/1.1 404 Not Found
Content-Type: application/problem+json

{
  "type": "https://example.com/errors/not-found",
  "title": "Resource Not Found",
  "status": 404,
  "detail": "Product with identifier '999' was not found.",
  "instance": "/api/products/999",
  "errorCode": "NOT_FOUND",
  "traceId": "00-abc123def456...",
  "timestamp": "2026-06-18T14:30:00.000Z"
}
```

Validation failure:

```json
HTTP/1.1 422 Unprocessable Entity
Content-Type: application/problem+json

{
  "type": "https://example.com/errors/validation",
  "title": "Validation Failed",
  "status": 422,
  "detail": "See the errors property for details.",
  "instance": "/api/products",
  "errors": {
    "Name": ["The Name field is required."],
    "Price": ["The Price field must be greater than 0."]
  },
  "errorCode": "VALIDATION_FAILED",
  "traceId": "00-789xyz...",
  "timestamp": "2026-06-18T14:31:00.000Z"
}
```

Server error:

```json
HTTP/1.1 500 Internal Server Error
Content-Type: application/problem+json

{
  "type": "https://httpstatuses.com/500",
  "title": "Internal Server Error",
  "status": 500,
  "detail": "An unexpected error occurred. Please try again later.",
  "traceId": "00-servercrash123...",
  "timestamp": "2026-06-18T14:32:00.000Z"
}
```

> [!summary] Section Summary
> - A production API combines `AddProblemDetails()`, `IExceptionHandler`, and `[ApiController]` for consistent error responses
> - Domain exceptions are converted to typed ProblemDetails with custom `errorCode` extensions
> - Validation failures return `ValidationProblemDetails` with field-level errors
> - Unexpected server errors return generic messages with a trace ID for log correlation
> - The `Content-Type` for error responses is `application/problem+json`
> - Use `[ProducesResponseType]` to document error responses in OpenAPI/Swagger

---

## Related Topics

- [[Exception Handling]] -- global exception handling that feeds into ProblemDetails responses
- [[ILogger and Logging]] -- structured logging for the server-side details behind each ProblemDetails traceId
- [[Middleware Overview]] -- how `UseExceptionHandler` and `UseStatusCodePages` fit in the pipeline
- [[Routing Overview]] -- how routing affects which endpoints produce ProblemDetails

---

## Further Reading

- [[API Versioning]] -- maintaining ProblemDetails consistency across API versions
- [[OpenAPI and Swagger]] -- documenting ProblemDetails responses in your API specification
- [[Content Negotiation]] -- how the framework decides between JSON and XML ProblemDetails
- [[Health Checks]] -- health check endpoints and their relationship to error monitoring

---

## Comprehensive Summary

> [!tip] Complete Summary
> **ProblemDetails** (RFC 9457, formerly RFC 7807) is the industry-standard format for HTTP API error responses. It provides five standard members: `type` (URI identifying the problem), `title` (generic summary), `status` (HTTP code), `detail` (occurrence-specific explanation), and `instance` (request reference). Custom extension members like `errorCode`, `traceId`, and `timestamp` add context without breaking the standard.
>
> **ASP.NET Core's built-in support** includes the `ProblemDetails` and `ValidationProblemDetails` classes, the `[ApiController]` attribute (which automatically returns ProblemDetails for errors and ValidationProblemDetails for validation failures), and `AddProblemDetails()` (.NET 7+) for global configuration.
>
> **`AddProblemDetails()`** with `CustomizeProblemDetails` is the single best place to add cross-cutting metadata (trace IDs, timestamps, node identifiers) to every error response. It applies to all ProblemDetails from any source: exception handlers, status code pages, `[ApiController]`, and manual creation.
>
> **`IProblemDetailsService`** enables ProblemDetails generation from middleware, minimal APIs, and anywhere outside controllers, while respecting global customizations. In minimal APIs, `Results.Problem()` and `Results.ValidationProblem()` provide convenient helpers.
>
> The most powerful pattern combines **`IExceptionHandler`** (.NET 8+) with `IProblemDetailsService`: domain exceptions (NotFoundException, ValidationException, etc.) are caught by specific handlers and converted to typed ProblemDetails with appropriate `type` URIs, status codes, and `errorCode` extensions. A fallback handler catches everything else and returns a generic 500 with no internal details.
>
> **Two audiences** are served simultaneously: machines use `type`, `status`, and `errorCode` for programmatic handling; humans read `title`, `detail`, and look up `traceId` in server logs. Keep `title` generic (same for every occurrence of the problem type) and `detail` specific (describes this particular failure).
>
> **Validation errors** use `ValidationProblemDetails` with an `errors` dictionary mapping field names to messages. `[ApiController]` generates this automatically for invalid model state before the action method runs. Customize via `ApiBehaviorOptions.InvalidModelStateResponseFactory`.
>
> The result is a consistent, documented, machine-readable error format across every endpoint in your API -- enabling generic client-side error handling and efficient debugging through trace ID correlation with [[ILogger and Logging|server-side logs]].