---
tags:
  - csharp
  - asp-net-core
  - middleware
  - custom
---

## Testing Custom Middleware

Testing middleware involves creating a `DefaultHttpContext`, a mock `RequestDelegate`, invoking the middleware, and asserting on the response.

### Test Setup Pattern

```csharp
using Microsoft.AspNetCore.Http;
using Xunit;

public class RequestTimingMiddlewareTests
{
    private readonly RequestTimingMiddleware _middleware;
    private readonly DefaultHttpContext _httpContext;
    private bool _nextWasCalled;

    public RequestTimingMiddlewareTests()
    {
        _httpContext = new DefaultHttpContext();
        // DefaultHttpContext gives us an in-memory response body
        // but we need to set up a real stream for header testing
        _httpContext.Response.Body = new MemoryStream();
        _nextWasCalled = false;
    }

    private RequestDelegate CreateMockNext(int delayMs = 50)
    {
        return async context =>
        {
            _nextWasCalled = true;
            // Simulate some work
            await Task.Delay(delayMs);
            context.Response.StatusCode = StatusCodes.Status200OK;
        };
    }

    [Fact]
    public async Task InvokeAsync_SetsResponseTimeHeader()
    {
        // Arrange
        var logger = NullLogger<RequestTimingMiddleware>.Instance;
        var next = CreateMockNext(delayMs: 100);
        var middleware = new RequestTimingMiddleware(next, logger);

        _httpContext.Request.Method = "GET";
        _httpContext.Request.Path = "/api/orders";

        // Act
        await middleware.InvokeAsync(_httpContext);

        // Fire the OnStarting callbacks (simulate response being sent)
        // In integration tests, Kestrel does this automatically.
        // For unit tests with DefaultHttpContext, we must invoke them manually.
        await _httpContext.Response.StartAsync();

        // Assert
        Assert.True(_nextWasCalled, "Middleware should call next delegate");
        Assert.True(
            _httpContext.Response.Headers.ContainsKey("X-Response-Time"),
            "Response should contain X-Response-Time header");

        var headerValue = _httpContext.Response.Headers["X-Response-Time"].ToString();
        Assert.Matches(@"\d+ms", headerValue);
    }

    [Fact]
    public async Task InvokeAsync_CallsNextMiddleware()
    {
        // Arrange
        var logger = NullLogger<RequestTimingMiddleware>.Instance;
        var next = CreateMockNext();
        var middleware = new RequestTimingMiddleware(next, logger);

        // Act
        await middleware.InvokeAsync(_httpContext);

        // Assert
        Assert.True(_nextWasCalled);
    }

    [Fact]
    public async Task InvokeAsync_MeasuresTimeGreaterThanZero()
    {
        // Arrange
        var logger = NullLogger<RequestTimingMiddleware>.Instance;
        var next = CreateMockNext(delayMs: 200);
        var middleware = new RequestTimingMiddleware(next, logger);

        // Act
        await middleware.InvokeAsync(_httpContext);
        await _httpContext.Response.StartAsync();

        // Assert
        var headerValue = _httpContext.Response.Headers["X-Response-Time"].ToString();
        var ms = int.Parse(headerValue.Replace("ms", ""));
        Assert.True(ms >= 100, $"Expected at least 100ms, got {ms}ms");
    }
}
```

### Testing Middleware That Short-Circuits

```csharp
public class ApiKeyMiddlewareTests
{
    [Fact]
    public async Task InvokeAsync_MissingApiKey_Returns401()
    {
        // Arrange
        var context = new DefaultHttpContext();
        context.Response.Body = new MemoryStream();
        context.Request.Path = "/api/orders";

        var nextCalled = false;
        RequestDelegate next = _ =>
        {
            nextCalled = true;
            return Task.CompletedTask;
        };

        var config = new ConfigurationBuilder()
            .AddInMemoryCollection(new Dictionary<string, string?>
            {
                ["Authentication:ApiKey"] = "test-key-12345"
            })
            .Build();

        var logger = NullLogger<ApiKeyMiddleware>.Instance;
        var middleware = new ApiKeyMiddleware(next, logger, config);

        // Act
        await middleware.InvokeAsync(context);

        // Assert
        Assert.Equal(StatusCodes.Status401Unauthorized, context.Response.StatusCode);
        Assert.False(nextCalled, "Next middleware should NOT be called");
    }

    [Fact]
    public async Task InvokeAsync_ValidApiKey_CallsNext()
    {
        // Arrange
        var context = new DefaultHttpContext();
        context.Request.Path = "/api/orders";
        context.Request.Headers["X-Api-Key"] = "test-key-12345";

        var nextCalled = false;
        RequestDelegate next = _ =>
        {
            nextCalled = true;
            return Task.CompletedTask;
        };

        var config = new ConfigurationBuilder()
            .AddInMemoryCollection(new Dictionary<string, string?>
            {
                ["Authentication:ApiKey"] = "test-key-12345"
            })
            .Build();

        var logger = NullLogger<ApiKeyMiddleware>.Instance;
        var middleware = new ApiKeyMiddleware(next, logger, config);

        // Act
        await middleware.InvokeAsync(context);

        // Assert
        Assert.True(nextCalled, "Next middleware should be called for valid key");
    }

    [Fact]
    public async Task InvokeAsync_HealthEndpoint_BypassesAuthentication()
    {
        // Arrange
        var context = new DefaultHttpContext();
        context.Request.Path = "/health";
        // No API key header set

        var nextCalled = false;
        RequestDelegate next = _ =>
        {
            nextCalled = true;
            return Task.CompletedTask;
        };

        var config = new ConfigurationBuilder()
            .AddInMemoryCollection(new Dictionary<string, string?>
            {
                ["Authentication:ApiKey"] = "test-key-12345"
            })
            .Build();

        var logger = NullLogger<ApiKeyMiddleware>.Instance;
        var middleware = new ApiKeyMiddleware(next, logger, config);

        // Act
        await middleware.InvokeAsync(context);

        // Assert
        Assert.True(nextCalled, "Health endpoint should bypass API key check");
    }
}
```

> [!tip]
> For more realistic testing, use `WebApplicationFactory<T>` from `Microsoft.AspNetCore.Mvc.Testing` to create an integration test that exercises the full pipeline including middleware ordering, DI, and real HTTP semantics. Unit tests with `DefaultHttpContext` are faster but do not test the pipeline wiring.

> [!summary] Section Summary
> Test middleware by constructing a `DefaultHttpContext`, creating a mock `RequestDelegate` (a simple lambda), invoking `InvokeAsync`, and asserting on the response status code, headers, and whether `next` was called. Use `Response.StartAsync()` to trigger `OnStarting` callbacks in unit tests. For full pipeline testing, use `WebApplicationFactory<T>`.
