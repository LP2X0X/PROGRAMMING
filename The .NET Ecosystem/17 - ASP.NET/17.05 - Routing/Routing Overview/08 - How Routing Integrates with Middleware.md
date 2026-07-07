---
tags:
  - csharp
  - asp-net-core
  - routing
  - fundamentals
---


Routing is not a standalone system -- it is deeply integrated into the ASP.NET Core **middleware pipeline**. Understanding this integration is essential for building correct applications.

### The Middleware Pipeline

ASP.NET Core processes each request through an ordered sequence of middleware components. Each middleware can:
- Handle the request and short-circuit the pipeline
- Pass the request to the next middleware
- Run logic before and/or after the next middleware

### Routing's Place in the Pipeline

```csharp
var app = builder.Build();

// Middleware that runs BEFORE routing (no endpoint info available)
app.UseHttpsRedirection();
app.UseStaticFiles();      // Can short-circuit for static files

// ROUTING: selects the endpoint
app.UseRouting();

// Middleware between routing and endpoint execution
// These CAN inspect the selected endpoint
app.UseCors();
app.UseAuthentication();
app.UseAuthorization();
app.UseRateLimiter();

// ENDPOINT EXECUTION: runs the selected endpoint
app.MapControllers();
app.MapRazorPages();
app.MapGet("/hello", () => "Hello World");
```

### The Three Zones

Think of the pipeline as three zones:

| Zone | Position | Endpoint Available? | Use For |
|---|---|---|---|
| Pre-routing | Before `UseRouting()` | No | Static files, HTTPS redirect, exception handling |
| Between routing and execution | After `UseRouting()`, before `Map*` | Yes (selected but not executed) | Auth, CORS, rate limiting -- anything that needs to inspect the endpoint |
| Post-routing | Inside `Map*` handlers | Yes (executing) | The actual request handling |

### Implicit UseRouting

> [!tip] Practical Tip
> In .NET 6+ with minimal hosting, `UseRouting()` is called **implicitly** if you do not call it yourself. The framework inserts it at the correct position. However, if you need middleware to run between routing and endpoint execution (which is common), you should call `UseRouting()` explicitly to control the position.

### Accessing the Selected Endpoint in Middleware

Any middleware after `UseRouting()` can inspect the selected endpoint:

```csharp
app.Use(async (context, next) =>
{
    var endpoint = context.GetEndpoint();

    if (endpoint != null)
    {
        // The endpoint was selected -- inspect its metadata
        var routeName = endpoint.DisplayName;
        var requiresAuth = endpoint.Metadata.GetMetadata<AuthorizeAttribute>();
    }
    else
    {
        // No endpoint matched this request
    }

    await next(context);
});
```

> [!warning] Common Misconception
> `UseRouting()` does **not** execute the endpoint. It only selects which endpoint matches. The endpoint runs later, at the end of the pipeline. If no endpoint matches, `context.GetEndpoint()` returns `null` and the request falls through to a 404.

> [!summary] Section Summary
> - The middleware pipeline has three zones: pre-routing, between routing and execution, and post-routing.
> - `UseRouting()` selects the endpoint; `Map*` methods execute it at the end of the pipeline.
> - Middleware between the two phases can inspect the selected endpoint and its metadata.
> - In .NET 6+, `UseRouting()` is implicit but should be called explicitly when you need to control middleware ordering.
> - `context.GetEndpoint()` returns the selected endpoint or `null` if no route matched.
