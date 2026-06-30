---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - lifetimes
---

## Why HttpClient Uses Singleton Registration

### The Socket Exhaustion Problem

Creating `HttpClient` instances directly (or registering them as Transient) causes a well-known problem: **socket exhaustion**. Each `HttpClient` instance manages its own connection pool. When you dispose an `HttpClient`, the underlying sockets enter a `TIME_WAIT` state and are not immediately available for reuse. Under load, this exhausts the available sockets on the machine.

```csharp
// BAD: Do not do this
public class OrderApiClient
{
    public async Task<Order> GetOrderAsync(int orderId)
    {
        // Creates a new HttpClient (and connection pool) every call!
        using var client = new HttpClient();
        var response = await client.GetAsync($"https://api.example.com/orders/{orderId}");
        return await response.Content.ReadFromJsonAsync<Order>();
    }
}
```

### The Solution: IHttpClientFactory

`IHttpClientFactory` manages a pool of `HttpMessageHandler` instances (the underlying connection handlers), reusing them across `HttpClient` instances. The `HttpClient` instances themselves are transient (lightweight wrappers), but the handlers are pooled and recycled.

```csharp
// Registration in Program.cs
builder.Services.AddHttpClient<IOrderApiClient, OrderApiClient>(client =>
{
    client.BaseAddress = new Uri("https://api.example.com");
    client.DefaultRequestHeaders.Add("Accept", "application/json");
    client.Timeout = TimeSpan.FromSeconds(30);
});
```

```csharp
// The typed client
public class OrderApiClient : IOrderApiClient
{
    private readonly HttpClient _client;

    // HttpClient is injected by the factory -- handler is pooled
    public OrderApiClient(HttpClient client)
    {
        _client = client;
    }

    public async Task<Order> GetOrderAsync(int orderId)
    {
        var response = await _client.GetAsync($"/orders/{orderId}");
        response.EnsureSuccessStatusCode();
        return await response.Content.ReadFromJsonAsync<Order>();
    }
}
```

> [!ad-note]
> When you use `AddHttpClient<TClient, TImplementation>()`, the typed client (`OrderApiClient`) is registered as **Transient** with the container, but the underlying `HttpMessageHandler` is managed by the factory with a default lifetime of 2 minutes before recycling. This gives you the best of both worlds: fresh client instances (no stale state) with pooled connections (no socket exhaustion).

> [!tip]
> Always use `IHttpClientFactory` (via `AddHttpClient`) instead of `new HttpClient()`. This applies to all HTTP calls in ASP.NET Core, whether to external APIs, microservices, or third-party providers. See [[Registration Patterns]] for more factory patterns.

> [!summary] Section Summary
> - Creating `HttpClient` directly or as Transient causes socket exhaustion under load.
> - `IHttpClientFactory` pools `HttpMessageHandler` instances while keeping `HttpClient` wrappers transient.
> - Use `AddHttpClient<TClient, TImplementation>()` for typed clients with pooled connections.
> - The default handler lifetime is 2 minutes before recycling, balancing connection reuse and DNS changes.
> - Never use `new HttpClient()` in a constructor or method -- always go through the factory.
