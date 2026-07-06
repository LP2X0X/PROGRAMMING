---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - pitfalls
---

## Thread Safety with Singletons

Singleton services are created once and shared across **all** HTTP requests on **all** threads simultaneously. Any mutable state within a Singleton must be explicitly protected against concurrent access.

### The Buggy Singleton

```csharp
// RateLimiter.cs -- NOT THREAD-SAFE
public class RateLimiter : IRateLimiter
{
    // A regular Dictionary is NOT thread-safe for concurrent writes.
    private readonly Dictionary<string, int> _requestCounts = new();
    private DateTime _windowStart = DateTime.UtcNow;

    public bool IsAllowed(string clientIp)
    {
        ResetWindowIfExpired();

        // RACE CONDITION: Two threads for the same IP can both read 0,
        // both increment to 1, and the client gets double the allowed requests.
        if (_requestCounts.ContainsKey(clientIp))
        {
            _requestCounts[clientIp]++;
        }
        else
        {
            _requestCounts[clientIp] = 1; // Can throw on concurrent Add
        }

        return _requestCounts[clientIp] <= 100;
    }

    private void ResetWindowIfExpired()
    {
        if (DateTime.UtcNow - _windowStart > TimeSpan.FromMinutes(1))
        {
            _requestCounts.Clear(); // Can throw if another thread is iterating
            _windowStart = DateTime.UtcNow;
        }
    }
}
```

This code will cause intermittent `InvalidOperationException` ("Collection was modified") crashes under load, plus incorrect counting due to race conditions.

### The Fix with ConcurrentDictionary

```csharp
// RateLimiter.cs -- THREAD-SAFE
public class RateLimiter : IRateLimiter
{
    private readonly ConcurrentDictionary<string, int> _requestCounts = new();
    private DateTime _windowStart = DateTime.UtcNow;
    private readonly object _windowLock = new();

    public bool IsAllowed(string clientIp)
    {
        ResetWindowIfExpired();

        // AddOrUpdate is atomic -- no race condition
        var count = _requestCounts.AddOrUpdate(
            clientIp,
            addValue: 1,
            updateValueFactory: (key, existing) => existing + 1);

        return count <= 100;
    }

    private void ResetWindowIfExpired()
    {
        lock (_windowLock)
        {
            if (DateTime.UtcNow - _windowStart > TimeSpan.FromMinutes(1))
            {
                _requestCounts.Clear();
                _windowStart = DateTime.UtcNow;
            }
        }
    }
}
```

> [!caution] Not Just Dictionaries
> Any mutable state in a Singleton is a potential concurrency bug. This includes:
> - `List<T>`, `Dictionary<TKey, TValue>`, `HashSet<T>` -- use their `Concurrent` counterparts
> - Simple counters (`int _count++`) -- use `Interlocked.Increment`
> - Boolean flags -- use `volatile` or `Interlocked.Exchange`
> - Any shared object being mutated -- protect with `lock` or `SemaphoreSlim` for async code

> [!tip]
> The safest Singleton is a **stateless** Singleton or one with only immutable/read-only state. If your Singleton needs mutable state, consider whether the state should instead live in a Scoped service (per-request), a distributed cache, or a database.

> [!summary] Section Summary
> - Singleton services are shared across all threads simultaneously -- mutable state must be protected
> - Use `ConcurrentDictionary`, `ConcurrentQueue`, etc. instead of their non-thread-safe counterparts
> - Protect compound operations (check-then-act) with `lock` or atomic operations like `Interlocked`
> - The safest approach is to keep Singletons stateless or immutable whenever possible
> - Thread safety bugs in Singletons are intermittent and extremely hard to reproduce in testing
