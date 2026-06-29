---
tags:
  - csharp
  - asp-net-core
  - routing
  - fundamentals
---


**.NET Core 3.0** introduced **[[Endpoint Routing]]** -- a unified routing system that decoupled route matching from endpoint execution. This is the system all modern ASP.NET Core applications use.

### The Two-Phase Model

Endpoint routing splits routing into two distinct phases:

1. **Route matching** (`UseRouting()`) -- examines the incoming URL and selects the best matching endpoint. The selected endpoint is attached to the `HttpContext` but *not yet executed*.

2. **Endpoint execution** (the terminal middleware like `MapControllers()`) -- runs the selected endpoint's request delegate.

```csharp
var app = builder.Build();

app.UseRouting();           // Phase 1: Select endpoint
app.UseAuthentication();    // Runs AFTER endpoint is selected
app.UseAuthorization();     // Can inspect the selected endpoint's metadata
// Phase 2: Execute the selected endpoint (implicitly at the end of the pipeline)
app.MapControllers();
app.MapRazorPages();
```

> [!ad-note] Key Insight
> The power of this two-phase model is that **middleware between `UseRouting()` and the endpoint execution can inspect the selected endpoint's metadata**. Authorization middleware, for example, can check whether the endpoint requires authentication *before* the endpoint code runs.

### Why This Matters

Before endpoint routing:
- Authorization had to run *inside* MVC filters, not in the middleware pipeline
- CORS decisions could not be made until MVC processed the request
- There was no way for generic middleware to know what code would eventually handle the request

After endpoint routing:
- Any middleware can call `context.GetEndpoint()` to inspect the selected endpoint
- Authorization, CORS, rate limiting, and other middleware work uniformly across all endpoint types
- The same routing system serves controllers, Razor Pages, minimal APIs, gRPC, and SignalR

> [!summary] Section Summary
> - Endpoint routing (since .NET Core 3.0) unifies all routing into a two-phase model.
> - Phase 1 (`UseRouting()`) selects the endpoint; Phase 2 executes it.
> - Middleware between the two phases can inspect endpoint metadata (authorization, CORS, etc.).
> - This design enables cross-cutting concerns to work uniformly across all endpoint types.
