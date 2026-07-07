---
tags:
  - csharp
  - asp-net-core
  - minimal-apis
  - web-api
---


You can access the raw HTTP request and response objects by declaring them as handler parameters.

### HttpContext, HttpRequest, HttpResponse

```csharp
// Full HttpContext access
app.MapGet("/context", (HttpContext context) =>
{
    var userAgent = context.Request.Headers.UserAgent.ToString();
    var clientIp = context.Connection.RemoteIpAddress?.ToString();
    return Results.Ok(new { userAgent, clientIp });
});

// Just the request
app.MapGet("/request-info", (HttpRequest request) =>
{
    return Results.Ok(new
    {
        Method = request.Method,
        Path = request.Path.Value,
        QueryString = request.QueryString.Value,
        ContentType = request.ContentType,
        Host = request.Host.Value
    });
});

// Direct response writing
app.MapGet("/custom-response", async (HttpResponse response) =>
{
    response.StatusCode = 200;
    response.ContentType = "text/plain";
    await response.WriteAsync("Custom response written directly");
});
```

### CancellationToken

The `CancellationToken` parameter is bound to `HttpContext.RequestAborted`, which fires when the client disconnects:

```csharp
app.MapGet("/products/export", async (
    IProductService service,
    CancellationToken cancellationToken) =>
{
    var products = await service.ExportAllAsync(cancellationToken);
    return Results.Ok(products);
});
```

### ClaimsPrincipal

Access the authenticated user directly:

```csharp
app.MapGet("/profile", (ClaimsPrincipal user) =>
{
    var userId = user.FindFirstValue(ClaimTypes.NameIdentifier);
    var email = user.FindFirstValue(ClaimTypes.Email);
    return Results.Ok(new { userId, email });
}).RequireAuthorization();
```

### Reading Request Headers

```csharp
app.MapGet("/products", (
    [FromHeader(Name = "X-Correlation-Id")] string? correlationId,
    [FromHeader(Name = "Accept-Language")] string? language) =>
{
    return Results.Ok(new { correlationId, language });
});
```

> [!summary] Section Summary
> Special types like `HttpContext`, `HttpRequest`, `HttpResponse`, `CancellationToken`, and `ClaimsPrincipal` are automatically available as handler parameters. They provide direct access to the HTTP pipeline without any explicit binding attributes.
