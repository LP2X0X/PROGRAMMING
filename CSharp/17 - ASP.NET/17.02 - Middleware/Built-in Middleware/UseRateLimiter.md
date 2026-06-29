---
tags:
  - csharp
  - asp-net-core
  - middleware
  - built-in
---

## UseRateLimiter

**`UseRateLimiter`** (introduced in **.NET 7**) provides built-in rate limiting to protect your application from excessive requests. It supports four algorithms: **fixed window**, **sliding window**, **token bucket**, and **concurrency limiter**.

### Service Registration

```csharp
using System.Threading.RateLimiting;

builder.Services.AddRateLimiter(options =>
{
    // Global rejection behavior
    options.RejectionStatusCode = StatusCodes.Status429TooManyRequests;

    // When a request is rejected, add Retry-After header
    options.OnRejected = async (context, cancellationToken) =>
    {
        context.HttpContext.Response.Headers.RetryAfter = "60";
        await context.HttpContext.Response.WriteAsJsonAsync(new ProblemDetails
        {
            Status = 429,
            Title = "Too Many Requests",
            Detail = "Rate limit exceeded. Please retry after 60 seconds."
        }, cancellationToken);
    };
});
```

### Fixed Window Limiter

Allows a fixed number of requests within a time window. The window resets completely when the time expires.

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("FixedPolicy", opt =>
    {
        opt.PermitLimit = 100;
        opt.Window = TimeSpan.FromMinutes(1);
        opt.QueueProcessingOrder = QueueProcessingOrder.OldestFirst;
        opt.QueueLimit = 10; // queue 10 extra requests instead of rejecting
    });
});
```

> [!info]
> **Fixed window** is the simplest algorithm. The downside is the "burst at boundary" problem: if a client sends 100 requests at the end of window 1 and 100 more at the start of window 2, they effectively send 200 requests in a very short period. Use **sliding window** to mitigate this.

### Sliding Window Limiter

Divides the window into segments and smoothly slides the limit, preventing boundary burst problems.

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddSlidingWindowLimiter("SlidingPolicy", opt =>
    {
        opt.PermitLimit = 100;
        opt.Window = TimeSpan.FromMinutes(1);
        opt.SegmentsPerWindow = 6; // 10-second segments
        opt.QueueProcessingOrder = QueueProcessingOrder.OldestFirst;
        opt.QueueLimit = 5;
    });
});
```

### Token Bucket Limiter

Tokens are added to a bucket at a fixed rate. Each request consumes one token. When the bucket is empty, requests are rejected. This naturally allows short bursts while enforcing an average rate.

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddTokenBucketLimiter("TokenBucketPolicy", opt =>
    {
        opt.TokenLimit = 50;           // max tokens in bucket
        opt.ReplenishmentPeriod = TimeSpan.FromSeconds(10);
        opt.TokensPerPeriod = 10;      // add 10 tokens every 10 seconds
        opt.QueueProcessingOrder = QueueProcessingOrder.OldestFirst;
        opt.QueueLimit = 5;
        opt.AutoReplenishment = true;
    });
});
```

### Concurrency Limiter

Limits the number of **concurrent** requests rather than requests over time. Useful for protecting resource-intensive endpoints.

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddConcurrencyLimiter("ConcurrencyPolicy", opt =>
    {
        opt.PermitLimit = 20;          // max 20 concurrent requests
        opt.QueueProcessingOrder = QueueProcessingOrder.OldestFirst;
        opt.QueueLimit = 10;
    });
});
```

### Applying Rate Limiters to Endpoints

```csharp
// Program.cs -- middleware
app.UseRateLimiter();

// Apply to minimal API endpoints
app.MapGet("/api/orders", () => Results.Ok())
    .RequireRateLimiting("SlidingPolicy");

// Apply to controllers via attribute
[EnableRateLimiting("TokenBucketPolicy")]
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    [DisableRateLimiting] // Exempt this endpoint
    [HttpGet("health")]
    public IActionResult HealthCheck() => Ok();

    [HttpPost]
    public IActionResult CreateOrder(OrderRequest request)
    {
        // Rate-limited by TokenBucketPolicy
        return Ok();
    }
}
```

### Rate Limiting by Client IP

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddPolicy("PerClientPolicy", context =>
    {
        var clientIp = context.Connection.RemoteIpAddress?.ToString() ?? "unknown";

        return RateLimitPartition.GetFixedWindowLimiter(clientIp, _ =>
            new FixedWindowRateLimiterOptions
            {
                PermitLimit = 60,
                Window = TimeSpan.FromMinutes(1)
            });
    });
});
```

### Algorithm Comparison

| Algorithm | Best For | Burst Handling | Complexity |
|---|---|---|---|
| Fixed Window | Simple rate limits | Poor (boundary burst) | Low |
| Sliding Window | Smooth rate enforcement | Good | Medium |
| Token Bucket | APIs with occasional burst tolerance | Excellent | Medium |
| Concurrency | Resource-intensive endpoints | N/A (concurrent, not time-based) | Low |

### When You Need It

APIs exposed to the public internet, multi-tenant applications, or any endpoint where you need to prevent abuse or protect backend resources.

### Gotchas

- You **must** call `builder.Services.AddRateLimiter()` to register the service -- `app.UseRateLimiter()` alone throws a runtime exception
- Rate limiters are **per-server instance** by default. In a load-balanced environment, each server tracks its own counts. For distributed rate limiting, use an external store (Redis) or an API gateway
- The `QueueLimit` determines how many excess requests wait in a queue instead of being immediately rejected. Setting it too high can cause memory pressure under heavy load
- Place `UseRateLimiter` **after** `UseRouting` so that per-endpoint policies can be resolved based on the matched endpoint

> [!summary] Section Summary
> `UseRateLimiter` (.NET 7+) provides four algorithms: fixed window, sliding window, token bucket, and concurrency limiter. Apply policies globally or per-endpoint. Rate limiting is per-server-instance by default -- use external stores for distributed scenarios. Always register the service with `AddRateLimiter` before using the middleware.
