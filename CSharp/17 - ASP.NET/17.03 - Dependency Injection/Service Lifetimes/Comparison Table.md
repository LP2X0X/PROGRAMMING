---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - lifetimes
---

## Comparison Table

| Lifetime | Created When | Disposed When | Same Within Request? | Same Across Requests? | Thread-Safe Required? | Typical Use Cases |
|---|---|---|---|---|---|---|
| **Transient** | Every time it is injected | When the scope (request) ends | No -- each injection gets a new instance | No | No (each consumer gets its own) | Validators, formatters, mappers, lightweight stateless services |
| **Scoped** | Once per scope (request) | When the scope (request) ends | Yes -- all injections share one instance | No -- each request gets its own | Only if used across async calls within the same request | DbContext, Unit of Work, repositories, request-specific state |
| **Singleton** | Once (on first request) | When the application shuts down | Yes | Yes -- same instance forever | Yes -- multiple threads access it concurrently | Caches, configuration, HttpClientFactory, loggers |

> [!ad-note]
> The "Disposed When" column only applies to services implementing `IDisposable` or `IAsyncDisposable`. The container calls `Dispose()` automatically at the appropriate time for each lifetime.

> [!summary] Section Summary
> - Transient: new per injection, disposed per scope, no thread safety needed.
> - Scoped: new per request, disposed per request, shared within request.
> - Singleton: created once, disposed at shutdown, shared everywhere, thread safety required.
> - The comparison table serves as a quick reference when choosing a lifetime.
> - Disposal is handled automatically by the container for all three lifetimes.
