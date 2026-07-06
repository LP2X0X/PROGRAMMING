---
tags:
  - csharp
  - asp-net-core
  - middleware
  - built-in
---

## UseRouting

**`UseRouting`** is responsible for matching incoming HTTP requests to **endpoints** defined in your application (controllers, minimal APIs, Razor Pages, SignalR hubs, gRPC services, etc.). It works in tandem with the **endpoint middleware** that actually executes the matched endpoint.

````ad-note
By default, `WebApplication` **automatically adds `RoutingMiddleware` at the start** of your pipeline — you don't see it, but it's there.

The problem: if you want something to run **before** routing, you can't, because routing is already first.

```
Default (implicit):
  [RoutingMiddleware] → [YourMiddleware] → [EndpointMiddleware]
                         ↑ too late — routing already happened
```

The fix: call `UseRouting()` yourself to **take control** of where it sits.
```csharp
app.UseStaticFiles();   // runs BEFORE routing
app.UseRouting();       // now YOU decide where routing goes
```
````

### How Routing Works

The routing system operates in two phases:
1. **`UseRouting`** -- examines the request URL and selects the best matching endpoint. It stores the matched endpoint in `HttpContext` via `IEndpointFeature`
2. **Endpoint execution** (e.g., `MapControllers`, `MapGet`) -- runs the selected endpoint's delegate

Any middleware placed **between** `UseRouting` and the endpoint execution can inspect the matched endpoint before it runs. This is how `UseAuthorization` knows which `[Authorize]` attributes to enforce.

### Configuration

```csharp
app.UseRouting();

// Middleware here can see which endpoint was matched
app.UseAuthentication();
app.UseAuthorization();

// Endpoints are executed here
app.MapControllers();
app.MapRazorPages();
```

### Inspecting the Matched Endpoint

```csharp
// Custom middleware that checks the matched endpoint
app.Use(async (context, next) =>
{
    var endpoint = context.GetEndpoint();
    if (endpoint != null)
    {
        var metadata = endpoint.Metadata.GetMetadata<IAuthorizeData>();
        // Log or inspect authorization requirements
    }
    await next(context);
});
```

### Implicit Routing in .NET 6+

In .NET 6 and later, `UseRouting` is implicitly called by `WebApplication` if you do not call it explicitly. However, calling it explicitly is necessary when you need to place middleware between routing and endpoint execution (which is common).

### When You Need It

Every ASP.NET Core application that uses endpoint routing (which is all modern ASP.NET Core apps).

### Gotchas

- `UseRouting` must come **before** `UseAuthorization` -- authorization needs to know the matched endpoint to evaluate `[Authorize]` attributes
- `UseRouting` must come **before** `UseCors` for CORS to apply policies based on endpoint metadata
- In .NET 6+, if you do not call `UseRouting` explicitly, the framework inserts it automatically at the beginning of the pipeline, which means middleware like `UseAuthentication` might not be able to see the matched endpoint

> [!summary] Section Summary
> `UseRouting` matches URLs to endpoints and makes the matched endpoint available to downstream middleware. It pairs with endpoint execution middleware (`MapControllers`, etc.). Explicit placement is important when other middleware (auth, CORS) needs to inspect endpoint metadata.
