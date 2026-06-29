---
tags: [csharp, asp-net-core, startup, program]
---


The `app.Map*()` methods define which code handles which URLs. This replaces the old `app.UseEndpoints(endpoints => { ... })` pattern.

### MVC Controllers

```csharp
// Map all controller routes using attribute routing
app.MapControllers();

// Map conventional MVC routes
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

// Map Razor Pages
app.MapRazorPages();
```

### Minimal API Endpoints

```csharp
// Simple GET endpoint
app.MapGet("/api/health", () => Results.Ok(new { Status = "Healthy" }));

// GET with route parameter and DI
app.MapGet("/api/orders/{id:int}", async (int id, IOrderService orderService) =>
{
    var order = await orderService.GetByIdAsync(id);
    return order is not null
        ? Results.Ok(order)
        : Results.NotFound();
});

// POST with request body
app.MapPost("/api/orders", async (CreateOrderRequest request, IOrderService orderService) =>
{
    var orderId = await orderService.CreateAsync(request);
    return Results.Created($"/api/orders/{orderId}", new { Id = orderId });
});

// Grouping endpoints
var orderGroup = app.MapGroup("/api/orders")
    .RequireAuthorization();

orderGroup.MapGet("/", async (IOrderService svc) => await svc.GetAllAsync());
orderGroup.MapGet("/{id:int}", async (int id, IOrderService svc) => await svc.GetByIdAsync(id));
orderGroup.MapPost("/", async (CreateOrderRequest req, IOrderService svc) => await svc.CreateAsync(req));
```

> [!ad-note] MapGroup
> `MapGroup` (introduced in .NET 7) lets you define a common prefix and apply shared filters (like authorization) to a group of endpoints. This keeps your `Program.cs` organized as the number of minimal API endpoints grows.

> [!summary] Section Summary
> - `MapControllers` / `MapControllerRoute` wire up MVC controllers.
> - `MapGet`, `MapPost`, `MapPut`, `MapDelete` define minimal API endpoints.
> - `MapGroup` organizes related endpoints under a common prefix with shared configuration.
> - Minimal APIs support parameter binding from route, query, body, and DI automatically.
