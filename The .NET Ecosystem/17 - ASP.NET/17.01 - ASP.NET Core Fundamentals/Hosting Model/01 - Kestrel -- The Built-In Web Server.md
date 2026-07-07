---
tags: [csharp, asp-net-core, hosting, kestrel]
---


Kestrel is the default, cross-platform web server included in every ASP.NET Core application. It runs on Windows, Linux, and macOS. When you call `WebApplication.CreateBuilder(args)`, Kestrel is automatically configured as the HTTP server unless you explicitly replace it.

### Why Kestrel Exists

Before ASP.NET Core, web applications on .NET were hosted exclusively through IIS on Windows. This created a hard dependency on a single operating system and a single web server. Kestrel was built to break that coupling:

- **Cross-platform**: runs anywhere .NET runs
- **High performance**: consistently ranks among the fastest web servers in TechEmpower benchmarks
- **Lightweight**: no dependency on OS-specific HTTP infrastructure
- **Embeddable**: the server lives inside your application process

### Minimal Kestrel Application

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/api/orders", () =>
{
    return Results.Ok(new { OrderId = 1042, Status = "Shipped" });
});

app.Run();
```

When this application starts, Kestrel begins listening on `http://localhost:5000` and `https://localhost:5001` by default.

### Kestrel Architecture

Kestrel processes requests through a pipeline:

1. **Transport layer** -- accepts TCP connections (uses Socket transport by default in .NET 6+)
2. **Connection middleware** -- handles TLS handshake, connection logging
3. **HTTP protocol parsing** -- parses HTTP/1.1, HTTP/2, and HTTP/3 frames
4. **Request delegation** -- hands the parsed request to the ASP.NET Core middleware pipeline

> [!tip] Performance Fact
> Kestrel uses asynchronous I/O throughout. It does not allocate a thread per connection. A single Kestrel instance can handle tens of thousands of concurrent connections efficiently using the thread pool and `async`/`await`.

> [!summary] Section Summary
> - Kestrel is the default, cross-platform web server built into ASP.NET Core
> - It replaced the IIS-only model from legacy ASP.NET Framework
> - Kestrel is lightweight, high-performance, and embeddable inside the application process
> - It supports HTTP/1.1, HTTP/2, and HTTP/3 out of the box
