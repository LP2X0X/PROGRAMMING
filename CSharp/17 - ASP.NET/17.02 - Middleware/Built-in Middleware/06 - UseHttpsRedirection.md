---
tags:
  - csharp
  - asp-net-core
  - middleware
  - built-in
---

## UseHttpsRedirection

**`UseHttpsRedirection`** automatically redirects HTTP requests to HTTPS by issuing a `307 Temporary Redirect` (or `301 Permanent Redirect`) response.

### Configuration

```csharp
// Program.cs -- services
builder.Services.AddHttpsRedirection(options =>
{
    options.RedirectStatusCode = StatusCodes.Status301MovedPermanently;
    options.HttpsPort = 5001;
});

// Program.cs -- middleware
app.UseHttpsRedirection();
```

### How the HTTPS Port is Determined

The middleware determines the HTTPS port in this order:
1. `HttpsRedirectionOptions.HttpsPort` (explicit configuration)
2. `ASPNETCORE_HTTPS_PORT` environment variable
3. `IServerAddressesFeature` (inspects the server's bound addresses)

If none of these provide a port, the middleware **logs a warning and does not redirect**. This is a common source of confusion in containerized environments.

### When You Need It

Any application that accepts both HTTP and HTTPS traffic and wants to enforce HTTPS.

### Gotchas

- In Docker/Kubernetes environments, the application often sits behind a reverse proxy that terminates TLS. In this case, HTTPS redirection can cause redirect loops. Use `UseForwardedHeaders` before `UseHttpsRedirection` to respect `X-Forwarded-Proto`
- If the HTTPS port cannot be determined, the middleware silently does nothing -- always verify it is working
- Use `307` (temporary) during development and testing; switch to `301` (permanent) only when you are confident HTTPS is fully configured

> [!ad-note]
> When behind a load balancer or reverse proxy, configure forwarded headers:
> ```csharp
> builder.Services.Configure<ForwardedHeadersOptions>(options =>
> {
>     options.ForwardedHeaders = ForwardedHeaders.XForwardedFor 
>         | ForwardedHeaders.XForwardedProto;
> });
> 
> app.UseForwardedHeaders();
> app.UseHttpsRedirection();
> ```

> [!summary] Section Summary
> `UseHttpsRedirection` enforces HTTPS by redirecting HTTP traffic. The HTTPS port must be discoverable or the middleware silently becomes a no-op. Be aware of reverse proxy scenarios that require `UseForwardedHeaders`.
