---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - problem-details
  - api
---


Here is a complete, production-ready API setup that uses ProblemDetails for all error responses with custom error codes and full traceability.

## Program.cs

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

## FallbackExceptionHandler.cs

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

## Example API Controller

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

## What the Client Sees

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
