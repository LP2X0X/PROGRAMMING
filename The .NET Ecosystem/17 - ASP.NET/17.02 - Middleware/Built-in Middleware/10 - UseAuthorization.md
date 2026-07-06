---
tags:
  - csharp
  - asp-net-core
  - middleware
  - built-in
---

## UseAuthorization

**`UseAuthorization`** evaluates authorization policies and attributes (`[Authorize]`, `[AllowAnonymous]`) against the authenticated user. It is the enforcement point that determines whether a request is allowed to proceed to the endpoint.

### How It Relates to Authentication

| Concern | Middleware | Question Answered |
|---|---|---|
| Authentication | `UseAuthentication` | "Who are you?" |
| Authorization | `UseAuthorization` | "Are you allowed to do this?" |

Authentication must run **before** authorization. If the user is not authenticated and the endpoint requires authorization, the middleware triggers an **authentication challenge** (e.g., redirect to login page or 401 response).

### Configuration

```csharp
// Program.cs -- services
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy =>
        policy.RequireRole("Admin"));

    options.AddPolicy("CanManageOrders", policy =>
        policy.RequireClaim("permission", "orders.manage"));

    // Fallback policy: require authenticated user for all endpoints by default
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

// Program.cs -- middleware
app.UseAuthentication();
app.UseAuthorization();
```

### Applying Policies

```csharp
[Authorize(Policy = "CanManageOrders")]
public class OrdersController : ControllerBase
{
    [AllowAnonymous]
    public IActionResult GetPublicCatalog() => Ok();

    public IActionResult CreateOrder(OrderRequest request) => Ok();
}

// Minimal API
app.MapDelete("/api/orders/{id}", (int id) => Results.NoContent())
    .RequireAuthorization("AdminOnly");
```

### When You Need It

Any application that needs to restrict access to endpoints based on user identity, roles, or claims.

### Gotchas

- `UseAuthorization` **must** come after `UseRouting` because it needs to know which endpoint was matched (and its metadata)
- `UseAuthorization` **must** come after `UseAuthentication` -- otherwise `HttpContext.User` is not yet populated
- Setting a `FallbackPolicy` that requires authentication means **every** endpoint requires authentication unless explicitly marked `[AllowAnonymous]`. This includes health checks and static files served after the authorization middleware
- `[AllowAnonymous]` on a controller method **overrides** `[Authorize]` on the controller class

> [!summary] Section Summary
> `UseAuthorization` enforces access control by evaluating policies against `HttpContext.User`. It depends on `UseAuthentication` running first and `UseRouting` having already matched an endpoint. Use `FallbackPolicy` carefully to avoid accidentally locking down public endpoints.
