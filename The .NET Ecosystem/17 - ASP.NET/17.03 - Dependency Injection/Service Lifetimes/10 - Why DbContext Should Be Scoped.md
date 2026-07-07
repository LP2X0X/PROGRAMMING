---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - lifetimes
---

## Why DbContext Should Be Scoped

EF Core registers `DbContext` as scoped by default when you call `AddDbContext<T>()`. This is not arbitrary -- it is the only safe choice for most applications.

### Why Not Singleton?

A singleton `DbContext` would be shared across all requests and all threads simultaneously. This causes multiple serious problems:

1. **DbContext is not thread-safe.** Concurrent requests modifying the same `DbContext` instance will throw exceptions or corrupt data.
2. **The change tracker grows forever.** Every entity loaded by any request stays tracked until the application restarts, consuming ever-increasing memory.
3. **Stale data.** Entities tracked from an earlier request do not reflect database changes made by other requests or external systems.
4. **Connection management.** A singleton context may hold database connections open far longer than necessary.

### Why Not Transient?

A transient `DbContext` gives a new instance to every injection point. Within a single request, this means:

1. **Inconsistent reads.** `OrderService` and `InventoryService` within the same request have different contexts. They may see different snapshots of the database.
2. **No shared transactions.** You cannot wrap operations across multiple services in a single transaction because they each have their own `DbContext` and therefore their own connection.
3. **Change tracker isolation.** An entity loaded in one service is unknown to another service's context, making cross-service operations awkward.

### Why Scoped Is Correct

Scoped gives you one `DbContext` per request:

- **Thread safety is handled** -- a single request is processed sequentially (or with controlled async/await), so the context is only accessed by one logical thread.
- **Change tracker is bounded** -- it accumulates entities only for the duration of one request, then gets disposed.
- **Shared state within a request** -- all services in the same request share the same context, enabling transactions and consistent reads.
- **Automatic disposal** -- the context is disposed when the request scope ends.

```csharp
// EF Core's default registration -- scoped
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));

// This is equivalent to:
builder.Services.AddScoped<AppDbContext>(sp =>
{
    var options = sp.GetRequiredService<DbContextOptions<AppDbContext>>();
    return new AppDbContext(options);
});
```

> [!danger]
> Never change `DbContext` to Singleton. It will appear to work in development with low traffic, then fail catastrophically under concurrent production load with `InvalidOperationException` or silent data corruption.

> [!summary] Section Summary
> - DbContext is registered as Scoped by default via `AddDbContext<T>()`.
> - Singleton DbContext breaks thread safety, leaks memory via the change tracker, and serves stale data.
> - Transient DbContext prevents shared transactions and causes inconsistent reads within a request.
> - Scoped DbContext provides one instance per request: thread-safe, bounded, and shared across services.
> - Never change DbContext's lifetime without understanding the full consequences.
