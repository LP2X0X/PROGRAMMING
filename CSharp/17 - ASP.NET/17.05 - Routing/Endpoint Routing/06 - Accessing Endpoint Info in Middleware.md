---
tags:
  - csharp
  - asp-net-core
  - routing
  - endpoints
---


Any middleware placed after `UseRouting()` can access the selected endpoint:

### Basic Access

```csharp
app.Use(async (context, next) =>
{
    var endpoint = context.GetEndpoint();

    if (endpoint is null)
    {
        // No route matched -- this request will likely 404
        Console.WriteLine("No matching endpoint");
    }
    else
    {
        // An endpoint was selected
        Console.WriteLine($"Matched: {endpoint.DisplayName}");

        // Access the route pattern (if RouteEndpoint)
        if (endpoint is RouteEndpoint routeEndpoint)
        {
            Console.WriteLine($"Pattern: {routeEndpoint.RoutePattern}");
        }
    }

    await next(context);
});
```

### Accessing Route Values

After routing, the matched route values (parameters extracted from the URL) are available:

```csharp
app.Use(async (context, next) =>
{
    var routeValues = context.Request.RouteValues;

    if (routeValues.TryGetValue("id", out var idValue))
    {
        Console.WriteLine($"Route parameter 'id' = {idValue}");
    }

    await next(context);
});
```

### Inspecting Metadata for Custom Logic

```csharp
// Custom attribute
[AttributeUsage(AttributeTargets.Method)]
public class FeatureFlagAttribute : Attribute
{
    public string FlagName { get; }
    public FeatureFlagAttribute(string flagName) => FlagName = flagName;
}

// Middleware that checks feature flags
app.Use(async (context, next) =>
{
    var endpoint = context.GetEndpoint();
    var featureFlag = endpoint?.Metadata.GetMetadata<FeatureFlagAttribute>();

    if (featureFlag is not null)
    {
        var featureService = context.RequestServices
            .GetRequiredService<IFeatureFlagService>();

        if (!featureService.IsEnabled(featureFlag.FlagName))
        {
            context.Response.StatusCode = 404;
            return; // Feature is disabled -- act as if endpoint doesn't exist
        }
    }

    await next(context);
});
```

> [!ad-note] Key Insight
> This pattern -- custom attributes as metadata, inspected by middleware -- is how ASP.NET Core implements authorization, CORS, rate limiting, and output caching internally. You can follow the same pattern for any cross-cutting concern: define an attribute, apply it to endpoints, and write middleware that reads it.

> [!summary] Section Summary
> - `context.GetEndpoint()` returns the selected endpoint (or `null` if no match).
> - `context.Request.RouteValues` provides extracted route parameter values.
> - `endpoint.Metadata.GetMetadata<T>()` retrieves specific metadata types.
> - Custom attributes + middleware inspection is the extensibility pattern for cross-cutting concerns.
