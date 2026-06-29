---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
  - request
---

## The Endpoint Routing Split

This is one of the most critical concepts to understand in ASP.NET Core middleware. **Endpoint routing is split into two phases**:

1. **`UseRouting()`** -- Selects which endpoint matches the request URL
2. **`Map*()`** / endpoint execution -- Executes the selected endpoint

Everything between these two phases has access to the **selected endpoint's metadata** (attributes, authorization policies, CORS policies) but runs BEFORE the endpoint executes.

```
UseRouting()          <-- Selects: "This request matches OrderController.GetOrder()"
    |
    v
UseAuthentication()   <-- Reads JWT token, sets HttpContext.User
    |
    v
UseAuthorization()    <-- Checks [Authorize(Policy = "OrderViewer")] on GetOrder
    |
    v
MapControllers()      <-- Executes OrderController.GetOrder()
```

### Why the Split Matters

The split allows authorization to be **endpoint-aware**. Consider this controller:

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrderController : ControllerBase
{
    [HttpGet("{id}")]
    [Authorize(Policy = "OrderViewer")]
    public async Task<IActionResult> GetOrder(int id)
    {
        var order = await _orderService.GetByIdAsync(id);
        return Ok(order);
    }

    [HttpPost]
    [Authorize(Policy = "OrderManager")]
    public async Task<IActionResult> CreateOrder(CreateOrderRequest request)
    {
        var order = await _orderService.CreateAsync(request);
        return CreatedAtAction(nameof(GetOrder), new { id = order.Id }, order);
    }
}
```

When `UseRouting()` runs, it determines that a GET request to `/api/order/42` matches `GetOrder`. This information is stored in `HttpContext`. When `UseAuthorization()` runs next, it reads the endpoint metadata and finds the `[Authorize(Policy = "OrderViewer")]` attribute. It then checks whether the current user satisfies the "OrderViewer" policy.

Without the routing split, authorization would not know which specific action is being called, and therefore could not apply endpoint-specific policies.

### Accessing Endpoint Metadata Between the Split

Any middleware placed between `UseRouting()` and the endpoint can access endpoint metadata:

```csharp
app.UseRouting();

// Custom middleware that reads endpoint metadata
app.Use(async (context, next) =>
{
    var endpoint = context.GetEndpoint();
    if (endpoint != null)
    {
        var auditAttribute = endpoint.Metadata.GetMetadata<AuditLogAttribute>();
        if (auditAttribute != null)
        {
            Console.WriteLine($"Auditing request to: {endpoint.DisplayName}");
        }
    }

    await next(context);
});

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
```

> [!ad-note] Important Implementation Detail
> In modern ASP.NET Core (6+), `UseRouting()` is called implicitly by the framework if you do not call it yourself. However, understanding the split is still critical because the ordering of `UseAuthentication()` and `UseAuthorization()` relative to routing determines whether endpoint metadata is available during those checks.

> [!summary] Section Summary
> Endpoint routing is split into two phases: `UseRouting()` selects the endpoint, and `Map*()` executes it. Middleware between these two phases -- particularly authentication and authorization -- has access to endpoint metadata. This is how ASP.NET Core applies endpoint-specific authorization policies. The split is the reason authorization MUST come after routing.
