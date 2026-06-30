---
tags:
  - csharp
  - asp-net-core
  - routing
  - endpoints
---


Here is a complete `Program.cs` demonstrating a realistic application with mixed endpoint types:

```csharp
using Microsoft.AspNetCore.Diagnostics.HealthChecks;

var builder = WebApplication.CreateBuilder(args);

// -- Service Registration --
builder.Services.AddControllers();           // MVC controllers
builder.Services.AddRazorPages();             // Razor Pages
builder.Services.AddEndpointsApiExplorer();   // OpenAPI/Swagger
builder.Services.AddSwaggerGen();

builder.Services.AddAuthentication()
    .AddJwtBearer();

builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy =>
        policy.RequireRole("Admin"));
});

builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", policy =>
        policy.WithOrigins("https://myapp.com")
              .AllowAnyMethod()
              .AllowAnyHeader());
});

builder.Services.AddHealthChecks()
    .AddSqlServer(builder.Configuration.GetConnectionString("Default")!);

builder.Services.AddOutputCache(options =>
{
    options.AddPolicy("CacheProducts", builder =>
        builder.Expire(TimeSpan.FromMinutes(5)));
});

builder.Services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("Api", config =>
    {
        config.PermitLimit = 100;
        config.Window = TimeSpan.FromMinutes(1);
    });
});

var app = builder.Build();

// -- Middleware Pipeline --

// Pre-routing middleware
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
    app.UseSwagger();
    app.UseSwaggerUI();
}
else
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();

// Phase 1: Endpoint selection
app.UseRouting();

// Between-phase middleware (can inspect selected endpoint)
app.UseCors("AllowFrontend");
app.UseAuthentication();
app.UseAuthorization();
app.UseRateLimiter();
app.UseOutputCache();

// Phase 2: Endpoint registration and execution

// -- MVC Controllers (attribute-routed) --
app.MapControllers()
    .RequireRateLimiting("Api");

// -- Razor Pages --
app.MapRazorPages();

// -- Minimal API: Public endpoints --
var publicApi = app.MapGroup("/api/public")
    .AllowAnonymous()
    .WithTags("Public");

publicApi.MapGet("/status", () => new { status = "OK", timestamp = DateTime.UtcNow })
    .WithName("AppStatus");

publicApi.MapGet("/products", async (IProductService service) =>
    await service.GetFeaturedAsync())
    .CacheOutput("CacheProducts");

// -- Minimal API: Admin endpoints --
var adminApi = app.MapGroup("/api/admin")
    .RequireAuthorization("AdminOnly")
    .WithTags("Administration");

adminApi.MapGet("/stats", async (IStatsService service) =>
    await service.GetDashboardStatsAsync());

adminApi.MapPost("/cache/clear", (IOutputCacheStore cache) =>
{
    cache.EvictByTagAsync("products", default);
    return Results.Ok("Cache cleared");
});

// -- Health Checks --
app.MapHealthChecks("/health/live", new HealthCheckOptions
{
    Predicate = _ => false  // No dependency checks -- just "is the app alive?"
}).AllowAnonymous();

app.MapHealthChecks("/health/ready", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("ready")
}).RequireAuthorization("AdminOnly");

// -- SPA Fallback (must be last) --
app.MapFallbackToFile("index.html");

app.Run();
```

### What This Example Demonstrates

| Feature | Where |
|---|---|
| Two-phase model | `UseRouting()` -> auth middleware -> `Map*` calls |
| Attribute-routed controllers | `MapControllers()` |
| Razor Pages | `MapRazorPages()` |
| Minimal API endpoints | `MapGet()`, `MapPost()` |
| Endpoint groups | `MapGroup("/api/public")`, `MapGroup("/api/admin")` |
| Group-level authorization | `.RequireAuthorization("AdminOnly")` on admin group |
| Anonymous access | `.AllowAnonymous()` on public group and liveness check |
| Output caching | `.CacheOutput("CacheProducts")` on product list |
| Rate limiting | `.RequireRateLimiting("Api")` on controllers |
| Health checks | `/health/live` (public) and `/health/ready` (admin) |
| SPA fallback | `MapFallbackToFile("index.html")` as the last mapping |
| OpenAPI tags | `.WithTags()` for Swagger grouping |

> [!summary] Section Summary
> - A real-world `Program.cs` combines controllers, Razor Pages, minimal APIs, health checks, and fallbacks.
> - The middleware pipeline follows the three-zone pattern: pre-routing, between-phases, execution.
> - Endpoint groups organize minimal APIs with shared authorization, tags, and caching.
> - Health check endpoints are separated into liveness (public) and readiness (admin-only).
> - SPA fallback is registered last to catch unmatched routes.
