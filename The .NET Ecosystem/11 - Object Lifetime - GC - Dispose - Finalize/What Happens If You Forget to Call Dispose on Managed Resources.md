---
tags:
 - csharp
 - object-lifetime
 - disposable
---

A common misconception: "the GC will clean everything up eventually." The GC reclaims **memory** — the bytes on the heap. It does **not** call `Dispose()`. These are two completely different things.

## What the GC Does and Doesn't Do

| What | GC handles it? |
|---|---|
| Reclaiming heap memory | Yes (when object is unreachable) |
| Running finalizers | Yes (if one exists) |
| Running `Dispose()` | **No, never** |
| Breaking event subscriptions | **No** — and they prevent collection |
| Returning pooled resources | **No** |
| Stopping timers / background work | **No** |
| Flushing buffered writes | **No** |

The GC is a **memory manager**, not a **lifecycle manager**. `Dispose()` is lifecycle management. They solve different problems.

---

## Managed-Only Resources Have No Safety Net

Most managed-resource-holding classes **don't have a finalizer** — finalizers exist as a safety net for unmanaged resources. So if you forget `Dispose()` on a purely managed resource, the cleanup logic **simply never runs**.

This can actually be **worse** than forgetting `Dispose()` on an unmanaged wrapper — at least the unmanaged wrapper has a finalizer that eventually releases the OS handle.

| Aspect | Managed-only resource | Wraps unmanaged handles |
|---|---|---|
| Memory reclaimed by GC? | Yes, always | Yes (the managed wrapper) |
| Has a finalizer? | Should NOT have one | Should have one (safety net) |
| What happens without Dispose? | Logical leaks, delayed cleanup | Actual resource leak until finalizer runs |
| Finalizer saves you? | N/A — no finalizer | Eventually, but non-deterministically |

---

## Logical Leaks — The Real Consequence

The side effects that `Dispose()` would have handled do NOT happen automatically when the GC collects the object.

### Event Subscription Leak

The classic case where "the GC will clean it up" is **flat-out wrong**:

```csharp
class Subscriber : IDisposable
{
    private readonly Publisher _publisher;

    public Subscriber(Publisher publisher)
    {
        _publisher = publisher;
        _publisher.SomethingHappened += OnSomethingHappened;
    }

    private void OnSomethingHappened(object? sender, EventArgs e) { }

    public void Dispose()
    {
        _publisher.SomethingHappened -= OnSomethingHappened;
    }
}
```

If `Publisher` is long-lived, the event delegate holds a strong reference **back to** the subscriber. The GC sees it as a live object — it **cannot** collect it. Every subscriber you create without disposing accumulates forever, responding to events on views that no longer exist.

This is especially common in WPF/WinForms desktop development.

### HttpClient — Port Exhaustion

```csharp
foreach (var url in urls)
{
    var client = new HttpClient();
    var result = await client.GetStringAsync(url);
    // Sockets enter TIME_WAIT state, won't release for ~240 seconds
}
```

The underlying `SocketsHttpHandler` manages connection pools. Without `Dispose()`, connections linger in `TIME_WAIT`. Create enough fast enough and you get `SocketException: Address already in use`.

### DbConnection — Pool Starvation

```csharp
public async Task<List<Order>> GetOrdersLeaky()
{
    var connection = new MySqlConnection(connectionString);
    await connection.OpenAsync();
    // ... query ...
    return orders;
    // Connection never returned to pool!
}
```

`Dispose()` returns the connection to the pool. Without it, every call checks out another connection. Exhaust the pool (default ~100) and the next `OpenAsync` blocks or times out. The finalizer *will* eventually return it, but on the GC's schedule — not yours.

### CancellationTokenRegistration

```csharp
var registration = ct.Register(() => CleanupOnCancel());
await SomeOperationAsync();
// registration.Dispose() forgotten — callback stays rooted to the CancellationTokenSource
```

If the `CancellationTokenSource` is long-lived, every undisposed registration accumulates.

---

## Severity Spectrum

1. **Benign** — Object holds extra managed memory slightly longer. GC handles it. No real impact.
2. **Wasteful** — Timer keeps firing, background work continues unnecessarily. CPU/memory waste.
3. **Problematic** — Event subscriptions keep objects alive indefinitely. Memory grows over time.
4. **Critical** — Connection pools exhaust, sockets deplete. App fails under load.

---

See [[IDisposable]] for the dispose pattern implementation and the three scenarios (managed-only, unmanaged, derived classes).
