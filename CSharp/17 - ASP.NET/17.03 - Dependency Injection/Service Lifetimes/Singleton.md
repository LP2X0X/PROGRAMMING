---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - lifetimes
---

## Singleton

**Singleton** services are created once on first request and the same instance is reused for the entire lifetime of the application. Every request, every user, every thread -- they all share the same instance.

### Registration

```csharp
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();
```

You can also register a singleton with a pre-built instance:

```csharp
var cacheService = new MemoryCacheService();
builder.Services.AddSingleton<ICacheService>(cacheService);
```

Or with a factory:

```csharp
builder.Services.AddSingleton<ICacheService>(sp =>
{
    var config = sp.GetRequiredService<IConfiguration>();
    return new MemoryCacheService(config["CacheSize"]);
});
```

### Behavior

The container creates the instance when it is first requested (lazy initialization). From that point on, the same instance is returned everywhere:

```
Request 1: MemoryCacheService instance #1
Request 2: MemoryCacheService instance #1 (same!)
Request 3: MemoryCacheService instance #1 (same!)
... all the way until the application shuts down
```

### When to Use Singleton

Singleton is ideal for:
- **In-memory caches** -- shared across all requests
- **Configuration wrappers** -- application-wide settings that do not change per request
- **HttpClient factories** -- managed through `IHttpClientFactory`
- **Logging infrastructure** -- shared loggers
- **Services with expensive initialization** -- created once, amortized across all requests

```csharp
// Good candidates for Singleton
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();
builder.Services.AddSingleton<IAppConfiguration, AppConfiguration>();
builder.Services.AddSingleton<IEmailTemplateLoader, EmailTemplateLoader>();
```

> [!danger]
> Singleton services **must be thread-safe**. Multiple requests (threads) will access the same instance concurrently. If your singleton modifies shared state without synchronization, you will get race conditions and data corruption. Use `ConcurrentDictionary`, `lock`, `SemaphoreSlim`, or immutable data structures.

> [!warning] Common Misconception
> "I can inject a scoped service (like `DbContext`) into a singleton." You cannot do this safely. The scoped service would be resolved once during the singleton's creation and then held forever -- it becomes a **captive dependency**. See [[The Captive Dependency Problem]].

> [!summary] Section Summary
> - Singleton creates one instance for the entire application lifetime, shared across all requests and users.
> - Created lazily on first request (or eagerly if you provide a pre-built instance).
> - Must be thread-safe since concurrent requests share the same instance.
> - Ideal for caches, configuration, and expensive-to-create services.
> - Never inject scoped or transient services directly into a singleton.
