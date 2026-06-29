---
tags: [csharp, asp-net-core, hosting, kestrel]
---


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
