---
tags: [csharp, asp-net-core, hosting, kestrel]
---

# Hosting Model

> [!ad-note] Overview
> The **hosting model** in ASP.NET Core determines how your application listens for and processes HTTP requests. Unlike the legacy ASP.NET Framework -- which was tightly coupled to IIS -- ASP.NET Core is designed around a modular, cross-platform hosting architecture. The primary server is **Kestrel**, a lightweight, high-performance web server built on top of `libuv` (earlier versions) and now raw socket I/O. Understanding the hosting model is essential for making correct deployment decisions, configuring HTTPS, and tuning performance for production workloads.

## Table of Contents

- [[#Kestrel -- The Built-In Web Server]]
- [[#Kestrel as Edge Server vs Behind a Reverse Proxy]]
- [[#In-Process vs Out-of-Process Hosting on IIS]]
- [[#HTTP.sys -- Windows-Only Alternative]]
- [[#Self-Hosted Deployment]]
- [[#Docker Hosting]]
- [[#Configuring Kestrel]]
- [[#Port Configuration]]
- [[#Real-World Production Setup]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]

---

## Kestrel -- The Built-In Web Server

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

---

## Kestrel as Edge Server vs Behind a Reverse Proxy

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

---

## In-Process vs Out-of-Process Hosting on IIS

When deploying to IIS on Windows, ASP.NET Core supports two distinct hosting models.

### In-Process Hosting

The ASP.NET Core app runs **inside the IIS worker process** (`w3wp.exe`). IIS handles the HTTP connection, and the request is passed directly to the app through an in-process handler -- no network hop, no socket communication.

```xml
<!-- In web.config -->
<aspNetCore processPath="dotnet"
            arguments=".\OrderService.dll"
            stdoutLogEnabled="false"
            hostingModel="InProcess" />
```

### Out-of-Process Hosting

The ASP.NET Core app runs in a **separate process** alongside Kestrel. IIS acts as a reverse proxy, forwarding requests to Kestrel over a local socket or named pipe.

```xml
<!-- In web.config -->
<aspNetCore processPath="dotnet"
            arguments=".\OrderService.dll"
            stdoutLogEnabled="false"
            hostingModel="OutOfProcess" />
```

### Comparison Table

| Feature | In-Process | Out-of-Process |
|---|---|---|
| Hosting model value | `InProcess` | `OutOfProcess` |
| Process | `w3wp.exe` | `dotnet.exe` + `w3wp.exe` |
| Web server used | IIS HTTP Server | Kestrel (behind IIS as proxy) |
| Performance | Faster (no inter-process hop) | Slightly slower (proxy overhead) |
| Process management | IIS manages lifecycle | IIS starts/monitors the dotnet process |
| App isolation | Shares process with IIS | Separate process provides isolation |
| Cross-platform | Windows + IIS only | Can also run on Linux (without IIS) |
| Windows Auth | Native support | Supported via IIS proxy |
| Default in .NET 6+ | Yes | Must be explicitly configured |

> [!tip] When to Use Each
> **In-Process** is the default and recommended for most IIS deployments due to better performance. Choose **Out-of-Process** when you need process isolation (for example, if your app has memory leaks you want to contain) or when you want the same deployment model across IIS and Linux.

### Verifying the Hosting Model

You can check which model is active at runtime:

```csharp
app.MapGet("/api/hosting-info", (IWebHostEnvironment env) =>
{
    var server = app.Services.GetRequiredService<IServer>();
    return Results.Ok(new
    {
        ServerType = server.GetType().Name,
        Environment = env.EnvironmentName
    });
});
```

- In-process returns `IISHttpServer`
- Out-of-process returns `KestrelServer`

> [!summary] Section Summary
> - In-process hosting runs inside `w3wp.exe` with no inter-process communication overhead
> - Out-of-process hosting runs Kestrel in a separate `dotnet.exe` process with IIS as a reverse proxy
> - In-process is the default and faster; out-of-process provides better isolation
> - The `hostingModel` attribute in `web.config` controls which model is used

---

## HTTP.sys -- Windows-Only Alternative

HTTP.sys is a Windows-only web server built on the Windows HTTP Server API (`http.sys` kernel-mode driver). It is an alternative to Kestrel for scenarios that require OS-level features not available in Kestrel.

### When to Use HTTP.sys

- **Windows Authentication** (Negotiate/NTLM/Kerberos) without IIS
- **Port sharing** between multiple applications
- **Direct internet exposure** without a reverse proxy (HTTP.sys has built-in kernel-mode protection)
- **Response caching** at the kernel level
- **WebSocket** support with kernel-mode efficiency

### Configuring HTTP.sys

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.WebHost.UseHttpSys(options =>
{
    options.AllowSynchronousIO = false;
    options.Authentication.Schemes = AuthenticationSchemes.NTLM;
    options.Authentication.AllowAnonymous = false;
    options.MaxConnections = 500;
    options.MaxRequestBodySize = 30_000_000; // 30 MB
    options.UrlPrefixes.Add("https://orderservice.corp.local:443");
});

var app = builder.Build();

app.MapGet("/api/orders", (HttpContext context) =>
{
    var user = context.User.Identity?.Name;
    return Results.Ok(new { AuthenticatedUser = user });
});

app.Run();
```

> [!warning] Windows Only
> HTTP.sys is not available on Linux or macOS. If you need cross-platform deployment, you must use Kestrel. If you need Windows Authentication on Linux, consider using Kerberos with a SPNEGO middleware or an identity provider like Active Directory Federation Services (ADFS).

### HTTP.sys vs Kestrel

| Feature | Kestrel | HTTP.sys |
|---|---|---|
| Platform | Cross-platform | Windows only |
| Windows Auth | Not built-in | Native (NTLM, Kerberos) |
| Port sharing | No | Yes |
| Kernel-mode caching | No | Yes |
| Direct internet exposure | Possible but not recommended | Designed for it |
| HTTP/2 | Yes | Yes (Windows 10+/Server 2016+) |
| HTTP/3 | Yes (.NET 7+) | Limited |
| Performance (general) | Excellent | Good |

> [!summary] Section Summary
> - HTTP.sys is a Windows-only alternative to Kestrel built on the kernel-mode HTTP driver
> - Use it when you need Windows Authentication without IIS, port sharing, or kernel-mode caching
> - It is safe to expose directly to the internet due to its kernel-mode request filtering
> - For cross-platform scenarios, Kestrel remains the only option

---

## Self-Hosted Deployment

ASP.NET Core applications are self-contained executables that do not require an external web server to run. There are several ways to run them directly.

### Running with dotnet CLI

```bash
# Development
dotnet run --project ./src/OrderService/OrderService.csproj

# Production (from published output)
dotnet publish -c Release -o ./publish
dotnet ./publish/OrderService.dll
```

### Framework-Dependent vs Self-Contained

```bash
# Framework-dependent (requires .NET runtime on target machine)
dotnet publish -c Release -o ./publish

# Self-contained (bundles the runtime -- larger but no runtime dependency)
dotnet publish -c Release -r linux-x64 --self-contained -o ./publish

# Single-file deployment
dotnet publish -c Release -r linux-x64 --self-contained -p:PublishSingleFile=true -o ./publish
```

> [!example] Choosing a Deployment Strategy
> **Framework-dependent** is best when your servers already have the .NET runtime installed (common in enterprise environments). **Self-contained** is best for Docker images, edge deployments, or when you cannot control the target environment. **Single-file** is ideal for CLI tools or microservices where you want a single binary to deploy.

### Running as a Windows Service

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Host.UseWindowsService(options =>
{
    options.ServiceName = "OrderProcessingService";
});

var app = builder.Build();

app.MapGet("/api/health", () => Results.Ok("Healthy"));

app.Run();
```

Install and manage with `sc.exe`:

```bash
sc create OrderProcessingService binPath="C:\services\OrderService.exe"
sc start OrderProcessingService
sc stop OrderProcessingService
```

### Running as a Linux systemd Service

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Host.UseSystemd();

var app = builder.Build();
app.Run();
```

Create a unit file at `/etc/systemd/system/orderservice.service`:

```
[Unit]
Description=Order Processing Service
After=network.target

[Service]
Type=notify
ExecStart=/usr/bin/dotnet /opt/orderservice/OrderService.dll
WorkingDirectory=/opt/orderservice
Restart=always
RestartSec=10
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=ASPNETCORE_URLS=http://localhost:5000

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable orderservice
sudo systemctl start orderservice
sudo systemctl status orderservice
```

> [!summary] Section Summary
> - ASP.NET Core apps are self-hosted -- they carry their own web server (Kestrel)
> - `dotnet publish` produces deployable output in framework-dependent or self-contained modes
> - Windows services use `UseWindowsService()`; Linux daemons use `UseSystemd()`
> - Self-contained single-file deployment produces a single executable with no external dependencies

---

## Docker Hosting

Docker is the most common deployment target for ASP.NET Core applications in modern cloud environments. Microsoft provides official base images optimized for building and running .NET apps.

### Basic Dockerfile

```dockerfile
# Build stage
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

COPY ["OrderService.csproj", "./"]
RUN dotnet restore

COPY . .
RUN dotnet publish -c Release -o /app/publish

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app
COPY --from=build /app/publish .

EXPOSE 8080
ENV ASPNETCORE_URLS=http://+:8080
ENV ASPNETCORE_ENVIRONMENT=Production

ENTRYPOINT ["dotnet", "OrderService.dll"]
```

### Key Docker Images

| Image | Purpose | Size |
|---|---|---|
| `mcr.microsoft.com/dotnet/sdk:8.0` | Building and publishing | ~800 MB |
| `mcr.microsoft.com/dotnet/aspnet:8.0` | Running web apps | ~220 MB |
| `mcr.microsoft.com/dotnet/runtime:8.0` | Running console/worker apps | ~190 MB |
| `mcr.microsoft.com/dotnet/aspnet:8.0-alpine` | Minimal runtime (Alpine Linux) | ~110 MB |

> [!tip] Use Multi-Stage Builds
> Always use multi-stage builds. The SDK image is large (800 MB+) and contains compilers, NuGet tools, and other build-time dependencies. The final runtime image should use `aspnet` or `runtime` to keep the container small and reduce the attack surface.

### Docker Compose for Development

```yaml
version: '3.8'
services:
  orderservice:
    build: .
    ports:
      - "5000:8080"
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__InventoryDb=Server=db;Database=Inventory;User=sa;Password=Dev@Pass123
    depends_on:
      - db

  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=Dev@Pass123
    ports:
      - "1433:1433"
```

> [!ad-note] .NET 8 Port Change
> Starting with .NET 8, the default port in official Docker images changed from `80` to `8080`. If you are upgrading from .NET 6 or 7, update your `EXPOSE` directive and `ASPNETCORE_URLS` accordingly. Existing health checks, load balancer configurations, and Kubernetes readiness probes may also need updating.

> [!summary] Section Summary
> - Microsoft provides official Docker images for building (`sdk`) and running (`aspnet`) .NET applications
> - Multi-stage builds keep final images small by excluding build-time dependencies
> - .NET 8 images default to port 8080 instead of port 80
> - Docker Compose simplifies local development with database and service dependencies

---

## Configuring Kestrel

Kestrel exposes extensive configuration options for endpoints, limits, protocols, and certificates.

### Configuration via `appsettings.json`

```json
{
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://0.0.0.0:5000"
      },
      "Https": {
        "Url": "https://0.0.0.0:5001",
        "Certificate": {
          "Path": "/certs/orderservice.pfx",
          "Password": "cert-password-here"
        }
      }
    },
    "Limits": {
      "MaxConcurrentConnections": 1000,
      "MaxConcurrentUpgradedConnections": 100,
      "MaxRequestBodySize": 52428800,
      "KeepAliveTimeout": "00:02:00",
      "RequestHeadersTimeout": "00:00:30"
    }
  }
}
```

### Configuration via Code

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.WebHost.ConfigureKestrel(options =>
{
    // Listen on port 5000 for HTTP
    options.ListenAnyIP(5000);

    // Listen on port 5001 for HTTPS with a certificate
    options.ListenAnyIP(5001, listenOptions =>
    {
        listenOptions.UseHttps("/certs/orderservice.pfx", "cert-password");
    });

    // Configure limits
    options.Limits.MaxConcurrentConnections = 1000;
    options.Limits.MaxRequestBodySize = 50 * 1024 * 1024; // 50 MB
    options.Limits.KeepAliveTimeout = TimeSpan.FromMinutes(2);
    options.Limits.RequestHeadersTimeout = TimeSpan.FromSeconds(30);
    options.Limits.MinRequestBodyDataRate = new MinDataRate(
        bytesPerSecond: 100, gracePeriod: TimeSpan.FromSeconds(10));
});

var app = builder.Build();
app.Run();
```

### HTTP/2 and HTTP/3 Configuration

```csharp
builder.WebHost.ConfigureKestrel(options =>
{
    options.ListenAnyIP(5001, listenOptions =>
    {
        listenOptions.UseHttps();
        listenOptions.Protocols = HttpProtocols.Http1AndHttp2AndHttp3;
    });
});
```

> [!warning] HTTPS Certificate Management
> Never store certificate passwords in source control. Use environment variables, Azure Key Vault, AWS Secrets Manager, or the .NET Secret Manager for development. In production, prefer PEM certificates mounted as volumes or fetched from a secrets store at startup.

### Important Kestrel Limits

| Limit | Default | Description |
|---|---|---|
| `MaxConcurrentConnections` | Unlimited | Max simultaneous TCP connections |
| `MaxRequestBodySize` | 30 MB | Largest request body accepted |
| `KeepAliveTimeout` | 130 seconds | How long an idle keep-alive connection stays open |
| `RequestHeadersTimeout` | 30 seconds | Max time to receive request headers |
| `MaxRequestHeaderCount` | 100 | Max number of request headers |
| `MaxRequestHeadersTotalSize` | 32 KB | Max combined size of all request headers |
| `MaxRequestLineSize` | 8 KB | Max length of the HTTP request line |

> [!summary] Section Summary
> - Kestrel can be configured via `appsettings.json` or programmatically in `Program.cs`
> - Endpoints, TLS certificates, connection limits, and protocol versions are all configurable
> - HTTP/2 and HTTP/3 are supported and can be enabled per endpoint
> - Certificate passwords and secrets must never be stored in source control

---

## Port Configuration

ASP.NET Core provides multiple ways to configure which ports and URLs the application listens on. They follow a clear precedence order.

### Precedence Order (Highest to Lowest)

1. **Code** -- `options.ListenAnyIP(5000)` in `ConfigureKestrel`
2. **Command-line argument** -- `--urls "http://0.0.0.0:5000"`
3. **Environment variable** -- `ASPNETCORE_URLS=http://0.0.0.0:5000`
4. **`appsettings.json`** -- Kestrel endpoint configuration
5. **`launchSettings.json`** -- Development only (not deployed)

### Command-Line `--urls`

```bash
dotnet run --urls "http://0.0.0.0:5000;https://0.0.0.0:5001"
```

### Environment Variable

```bash
# Linux/macOS
export ASPNETCORE_URLS="http://+:5000;https://+:5001"

# Windows PowerShell
$env:ASPNETCORE_URLS = "http://+:5000;https://+:5001"
```

### launchSettings.json (Development Only)

```json
{
  "profiles": {
    "OrderService": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "applicationUrl": "https://localhost:7201;http://localhost:5201",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

> [!ad-note] `launchSettings.json` Is Not Deployed
> The `launchSettings.json` file lives under `Properties/` and is only used during local development with `dotnet run` or IDE launch. It is not included in published output. For staging and production, use environment variables or `appsettings.{Environment}.json`.

### URL Format Reference

| Format | Meaning |
|---|---|
| `http://localhost:5000` | Loopback only (IPv4 + IPv6) |
| `http://0.0.0.0:5000` | All IPv4 interfaces |
| `http://[::]:5000` | All IPv6 interfaces |
| `http://+:5000` | All interfaces (shorthand) |
| `http://*:5000` | All interfaces (alternative shorthand) |
| `https://+:5001` | All interfaces with HTTPS |

> [!summary] Section Summary
> - Ports can be configured via code, CLI arguments, environment variables, or config files
> - `ASPNETCORE_URLS` is the most common method for production deployments
> - `launchSettings.json` is development-only and not included in published output
> - Use `+` or `*` to listen on all network interfaces; use `localhost` to restrict to loopback

---

## Real-World Production Setup

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

---

## Comprehensive Summary

> [!tip] Complete Summary
> The ASP.NET Core hosting model is built around **Kestrel**, a cross-platform, high-performance web server embedded directly in the application. Kestrel can run as an edge server or behind a reverse proxy like **Nginx**, **IIS**, or **Apache** -- with the reverse proxy pattern being the recommended approach for production internet-facing workloads.
>
> When deploying to **IIS on Windows**, you choose between **in-process hosting** (faster, runs inside `w3wp.exe`) and **out-of-process hosting** (better isolation, runs Kestrel in a separate process). In-process is the default starting from .NET 6.
>
> For Windows-specific scenarios requiring **Windows Authentication** or **port sharing**, **HTTP.sys** provides a kernel-mode alternative to Kestrel -- though it sacrifices cross-platform compatibility.
>
> **Self-hosted deployments** use `dotnet publish` to produce framework-dependent or self-contained output, which can run as a **Windows Service** or a **Linux systemd daemon**. **Docker** is the dominant deployment model in cloud environments, using multi-stage builds with Microsoft's official `sdk` and `aspnet` images.
>
> **Kestrel configuration** covers endpoints, TLS certificates, connection limits, and protocol versions (HTTP/1.1, HTTP/2, HTTP/3). **Port configuration** follows a clear precedence: code overrides CLI arguments, which override environment variables, which override `appsettings.json`.
>
> A typical production setup places Kestrel behind Nginx with TLS termination at the proxy, forwarded headers middleware in the app, and the application listening only on the loopback interface. This architecture cleanly separates concerns and allows independent scaling and patching of each component.

---

## Related Topics

- [[ASP.NET Core Overview]]
- [[Project Structure]]
- [[Program.cs and Startup]]
- [[Environments]]
- [[Middleware Pipeline]]
- [[Configuration System]]
- [[Dependency Injection in ASP.NET Core]]
- [[Logging and Diagnostics]]
- [[Deployment Strategies]]
