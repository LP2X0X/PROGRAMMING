---
tags: [csharp, asp-net-core, hosting, kestrel]
---


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
