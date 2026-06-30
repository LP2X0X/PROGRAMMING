---
tags:
  - csharp
  - asp-net-core
  - routing
  - endpoints
---


ASP.NET Core includes built-in health check infrastructure that integrates directly with endpoint routing:

```csharp
// Register health check services
builder.Services.AddHealthChecks()
    .AddCheck("database", new SqlHealthCheck(connectionString))
    .AddCheck("redis", new RedisHealthCheck(redisConnection));

var app = builder.Build();

// Map health check endpoint
app.MapHealthChecks("/health");
```

### Customizing Health Check Responses

```csharp
app.MapHealthChecks("/health", new HealthCheckOptions
{
    ResponseWriter = async (context, report) =>
    {
        context.Response.ContentType = "application/json";
        var result = new
        {
            status = report.Status.ToString(),
            checks = report.Entries.Select(e => new
            {
                name = e.Key,
                status = e.Value.Status.ToString(),
                description = e.Value.Description
            })
        };
        await context.Response.WriteAsJsonAsync(result);
    }
});
```

### Health Checks with Authorization

```csharp
// Public liveness check
app.MapHealthChecks("/health/live")
    .AllowAnonymous();

// Detailed readiness check (requires auth)
app.MapHealthChecks("/health/ready", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("ready")
})
.RequireAuthorization("HealthCheckPolicy");
```

> [!tip] Practical Tip
> Container orchestrators like Kubernetes use health endpoints for liveness and readiness probes. Keep `/health/live` lightweight (just "is the process running?") and `/health/ready` for dependency checks (database, cache, external services). The liveness endpoint should almost never fail; the readiness endpoint tells the orchestrator whether to route traffic.

> [!summary] Section Summary
> - `MapHealthChecks("/health")` registers a health check endpoint.
> - Customize response format with `HealthCheckOptions.ResponseWriter`.
> - Separate liveness (`/health/live`) and readiness (`/health/ready`) endpoints for orchestrators.
> - Health check endpoints support authorization metadata like any other endpoint.
