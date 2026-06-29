---
tags:
  - csharp
  - asp-net-core
  - logging
  - ilogger
  - observability
---


A **correlation ID** (also called request ID or trace ID) is a unique identifier that follows a request through all the services and log entries it generates. This is essential for debugging in distributed systems where a single user action might hit multiple microservices.

## ASP.NET Core Built-in TraceIdentifier

ASP.NET Core automatically generates a `TraceIdentifier` for each request:

```csharp
// Every HttpContext has a TraceIdentifier
var traceId = context.TraceIdentifier;
// Example: "0HMVK2K9M8RHS:00000001"
```

## Custom Correlation ID Middleware

For cross-service tracing, use a custom correlation ID that can be passed between services:

```csharp
public class CorrelationIdMiddleware
{
    private const string CorrelationIdHeader = "X-Correlation-Id";
    private readonly RequestDelegate _next;

    public CorrelationIdMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Check if the caller (e.g., an upstream service) sent a correlation ID
        if (!context.Request.Headers.TryGetValue(
            CorrelationIdHeader, out var correlationId)
            || string.IsNullOrWhiteSpace(correlationId))
        {
            // Generate a new one if not provided
            correlationId = Guid.NewGuid().ToString("N");
        }

        // Store in HttpContext.Items for use in the current request
        context.Items["CorrelationId"] = correlationId.ToString();

        // Add to response headers so the client can use it for support requests
        context.Response.OnStarting(() =>
        {
            context.Response.Headers[CorrelationIdHeader] = correlationId;
            return Task.CompletedTask;
        });

        // Push to Serilog's LogContext so every log entry includes it
        using (LogContext.PushProperty("CorrelationId", correlationId.ToString()))
        {
            await _next(context);
        }
    }
}
```

## Propagating Correlation IDs to Downstream Services

When calling other services, forward the correlation ID:

```csharp
public class ExternalApiClient
{
    private readonly HttpClient _httpClient;
    private readonly IHttpContextAccessor _httpContextAccessor;

    public ExternalApiClient(
        HttpClient httpClient,
        IHttpContextAccessor httpContextAccessor)
    {
        _httpClient = httpClient;
        _httpContextAccessor = httpContextAccessor;
    }

    public async Task<Order> GetOrderAsync(int orderId)
    {
        var correlationId = _httpContextAccessor.HttpContext?
            .Items["CorrelationId"]?.ToString();

        var request = new HttpRequestMessage(HttpMethod.Get,
            $"/api/orders/{orderId}");

        if (correlationId is not null)
        {
            request.Headers.Add("X-Correlation-Id", correlationId);
        }

        var response = await _httpClient.SendAsync(request);
        response.EnsureSuccessStatusCode();
        return await response.Content.ReadFromJsonAsync<Order>();
    }
}
```

> [!tip]
> For production distributed tracing, consider using **OpenTelemetry** instead of custom correlation ID middleware. OpenTelemetry is an industry standard that provides distributed tracing, metrics, and logging -- with automatic context propagation using W3C Trace Context headers. Serilog integrates with OpenTelemetry via `Serilog.Enrichers.Span`.

> [!summary] Section Summary
> - Correlation IDs are unique identifiers that follow a request through all services and log entries
> - ASP.NET Core provides `HttpContext.TraceIdentifier` automatically, but custom IDs offer more control
> - Use middleware to extract or generate the ID, store it in `HttpContext.Items`, push it to `LogContext`
> - Forward the ID in outbound HTTP requests to downstream services
> - For production distributed systems, consider OpenTelemetry for industry-standard tracing
