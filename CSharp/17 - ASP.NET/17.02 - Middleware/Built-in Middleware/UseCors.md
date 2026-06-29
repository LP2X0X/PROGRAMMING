---
tags:
  - csharp
  - asp-net-core
  - middleware
  - built-in
---

## UseCors

**`UseCors`** adds **Cross-Origin Resource Sharing** headers to responses, allowing browsers to make requests from a different origin (domain, scheme, or port) than the one serving the application.

### Why CORS Exists

Browsers enforce the **Same-Origin Policy**, which blocks JavaScript from making requests to a different origin. CORS is the mechanism that relaxes this restriction in a controlled way. Without it, your API cannot be consumed by a frontend hosted on a different domain.

### Named Policy Configuration

```csharp
// Program.cs -- services
builder.Services.AddCors(options =>
{
    options.AddPolicy("OrderPortalPolicy", policy =>
    {
        policy.WithOrigins(
                "https://orderportal.example.com",
                "https://admin.example.com")
            .WithMethods("GET", "POST", "PUT", "DELETE")
            .WithHeaders("Content-Type", "Authorization")
            .AllowCredentials()
            .SetPreflightMaxAge(TimeSpan.FromMinutes(10));
    });

    options.AddPolicy("PublicApiPolicy", policy =>
    {
        policy.AllowAnyOrigin()
            .AllowAnyMethod()
            .AllowAnyHeader();
    });
});

// Program.cs -- middleware
app.UseRouting();
app.UseCors("OrderPortalPolicy"); // default policy

// Or apply per-endpoint:
app.MapControllers();
```

### Default Policy Configuration

```csharp
builder.Services.AddCors(options =>
{
    options.AddDefaultPolicy(policy =>
    {
        policy.WithOrigins("https://orderportal.example.com")
            .AllowAnyMethod()
            .AllowAnyHeader();
    });
});

// No policy name needed when using default
app.UseCors();
```

### Per-Endpoint CORS

```csharp
app.MapGet("/api/products", () => Results.Ok(products))
    .RequireCors("PublicApiPolicy");

// Or via attribute on controllers:
[EnableCors("OrderPortalPolicy")]
public class OrdersController : ControllerBase { }
```

### When You Need It

Whenever your API is consumed by a browser-based client hosted on a different origin.

### Gotchas

- `UseCors` must be placed **after** `UseRouting` and **before** `UseAuthorization` so that CORS preflight requests are handled before authorization rejects them
- `AllowAnyOrigin()` and `AllowCredentials()` **cannot be combined** -- the CORS specification forbids it. This will throw a runtime exception
- CORS is a **browser-enforced** security mechanism. Non-browser clients (Postman, `HttpClient`) ignore CORS entirely
- Forgetting to include `Authorization` in `WithHeaders` when your API requires bearer tokens causes preflight failures

> [!summary] Section Summary
> `UseCors` controls which origins can access your API from a browser. Use named policies for fine-grained control or a default policy for simplicity. Place it between `UseRouting` and `UseAuthorization`, and never combine `AllowAnyOrigin` with `AllowCredentials`.
