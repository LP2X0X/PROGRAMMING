---
tags: [csharp, asp-net-core, hosting, kestrel]
---


Kestrel can operate in two modes: as an **edge server** (directly exposed to the internet) or behind a **reverse proxy** (Nginx, IIS, Apache, YARP).

### Edge Server (Internet-Facing)

```
[Internet] --> [Kestrel] --> [ASP.NET Core App]
```

In this mode, Kestrel handles all incoming traffic directly. This is simpler but means Kestrel must handle everything: TLS termination, rate limiting, static file caching, and request filtering.

> [!warning] Security Consideration
> Running Kestrel as an edge server is supported but requires careful configuration. Kestrel does not provide all the hardening features that mature reverse proxies offer -- such as request buffering, connection draining, IP filtering, and advanced rate limiting. For internet-facing production workloads, a reverse proxy is strongly recommended.

### Behind a Reverse Proxy (Recommended for Production)

```
[Internet] --> [Nginx/IIS/Apache] --> [Kestrel] --> [ASP.NET Core App]
```

The reverse proxy handles:

- **TLS/SSL termination** -- offloads certificate management
- **Static file serving** -- serves images, CSS, and JS without hitting Kestrel
- **Load balancing** -- distributes requests across multiple Kestrel instances
- **Request buffering** -- protects against slow-client attacks (Slowloris)
- **URL rewriting and compression**

### Forwarded Headers

When behind a reverse proxy, the original client IP and protocol are lost. You must configure forwarded headers middleware:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardedHeaders =
        ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto;
    // Trust the proxy server
    options.KnownProxies.Add(IPAddress.Parse("10.0.0.1"));
});

var app = builder.Build();

app.UseForwardedHeaders();
app.UseHttpsRedirection();
app.UseAuthorization();

app.MapGet("/api/inventory/{sku}", (string sku) =>
{
    return Results.Ok(new { Sku = sku, Quantity = 47 });
});

app.Run();
```

> [!ad-note] Why Forwarded Headers Matter
> Without forwarded headers, `HttpContext.Connection.RemoteIpAddress` will return the proxy IP instead of the actual client IP. Logging, rate limiting, geo-blocking, and audit trails all depend on seeing the real client address.

| Scenario | Edge Server | Behind Reverse Proxy |
|---|---|---|
| TLS termination | Kestrel handles it | Proxy handles it |
| Static files | Kestrel serves them | Proxy serves them (faster) |
| Client IP visibility | Direct | Requires ForwardedHeaders |
| DDoS protection | Limited | Proxy provides buffering |
| Complexity | Lower | Higher (two processes) |
| Production recommendation | Internal APIs only | Internet-facing apps |

> [!summary] Section Summary
> - Kestrel can run as an edge server or behind a reverse proxy
> - Production deployments should use a reverse proxy for security, performance, and resilience
> - Forwarded headers middleware is required to preserve client IP and protocol information behind a proxy
