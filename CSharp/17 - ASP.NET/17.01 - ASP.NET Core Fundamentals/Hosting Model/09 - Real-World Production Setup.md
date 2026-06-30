---
tags: [csharp, asp-net-core, hosting, kestrel]
---


A typical production deployment for an internet-facing ASP.NET Core application uses Kestrel behind Nginx with TLS termination at the proxy layer.

### Architecture

```
[Client Browser]
       |
   [Nginx :443]  <-- TLS termination, static files, rate limiting
       |
   [Kestrel :5000]  <-- HTTP only (internal network)
       |
   [ASP.NET Core App]
       |
   [InventoryContext -> MariaDB/PostgreSQL]
```

### Nginx Configuration

```
server {
    listen 443 ssl http2;
    server_name orders.example.com;

    ssl_certificate     /etc/letsencrypt/live/orders.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/orders.example.com/privkey.pem;

    # Security headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains" always;

    # Static files served by Nginx
    location /static/ {
        root /var/www/orderservice;
        expires 30d;
    }

    # Proxy to Kestrel
    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection keep-alive;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}

server {
    listen 80;
    server_name orders.example.com;
    return 301 https://$host$request_uri;
}
```

### ASP.NET Core App Configuration

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<InventoryContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("InventoryDb")));

builder.Services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardedHeaders =
        ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto;
    options.KnownNetworks.Clear();
    options.KnownProxies.Clear();
});

var app = builder.Build();

app.UseForwardedHeaders();
app.UseHttpsRedirection();
app.UseAuthorization();

app.MapGet("/api/orders/{id}", async (int id, InventoryContext db) =>
{
    var order = await db.Orders.FindAsync(id);
    return order is not null ? Results.Ok(order) : Results.NotFound();
});

app.Run();
```

> [!example] Why This Architecture Works
> **Nginx** handles what it does best: TLS, static files, compression, and connection management. **Kestrel** handles what it does best: executing .NET code and processing dynamic requests. Neither component is asked to do work outside its strengths. This separation also means you can scale Kestrel instances independently, restart the app without dropping TLS sessions, and apply OS-level security patches to Nginx without redeploying the application.

### Production Checklist

- [ ] Kestrel listens on `http://localhost:5000` (loopback only -- not exposed to internet)
- [ ] Nginx terminates TLS with valid certificates (Let's Encrypt or CA-signed)
- [ ] Forwarded headers middleware is configured and trusted proxies are specified
- [ ] `ASPNETCORE_ENVIRONMENT` is set to `Production`
- [ ] Application runs as a non-root user via systemd or a container
- [ ] Health check endpoint is exposed for load balancer probes
- [ ] Logging is configured to write structured logs (Serilog, NLog) to a centralized system
- [ ] Request body size limits are set appropriately in both Nginx and Kestrel

> [!summary] Section Summary
> - The standard production architecture is Kestrel behind Nginx (or another reverse proxy) with TLS termination
> - Nginx handles TLS, static files, and connection management; Kestrel handles dynamic request processing
> - Forwarded headers must be configured to preserve client IP and protocol
> - Kestrel should listen on loopback only when behind a proxy
