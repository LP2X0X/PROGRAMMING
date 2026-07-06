---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - problem-details
  - api
---


The **`IProblemDetailsService`** interface (.NET 7+) lets you generate ProblemDetails programmatically from anywhere in your application -- not just from controllers. It is registered automatically when you call `AddProblemDetails()`.

```csharp
public interface IProblemDetailsService
{
    ValueTask WriteAsync(ProblemDetailsContext context);
}
```

## Using IProblemDetailsService in Middleware

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

## Using IProblemDetailsService in Minimal APIs

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
