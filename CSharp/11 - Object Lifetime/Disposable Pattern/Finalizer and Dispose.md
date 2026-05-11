---
tags:
 - csharp
 - object-lifetime
 - disposable
---

## Memory vs Resources — Why Dispose Exists

**Memory** — bytes on the managed heap. The GC tracks all of it and frees it automatically when no references remain. You never free managed memory yourself. `Dispose()` does **not** free memory — only the GC can do that.

**Resources** — anything the GC doesn't manage: OS file handles, sockets, DB connections, unmanaged memory (`Marshal.AllocHGlobal`), GPU buffers, distributed locks, etc. If you don't release these explicitly, they leak.

Unmanaged memory counts as a resource (not "memory" in the .NET sense) because it's allocated outside the managed heap — the GC can't see it or free it.

```
Managed heap object  → GC frees automatically (you can't control when)
Resource (OS handle) → YOU free via Dispose() (you must do this)
Unmanaged memory     → YOU free via Dispose() (GC doesn't know it exists)
```

`Dispose()` exists because resources are **scarce** — the OS might only give you ~16K file handles, or a DB might cap at 100 connections. You can't wait for the GC to maybe clean those up.

```ad-note
GC handles memory. Dispose handles resources. A managed resource is just a managed object that wraps something the GC can't see. The finalizer is a last-resort backup for when you forget to call Dispose.
```

---

## Finalizer vs Dispose — The Core Difference

**`Dispose()`** is the full package — clean up managed resources, unmanaged resources, everything. You call it explicitly when you're done with the object. It's deterministic: you decide exactly when cleanup happens.

**The finalizer (`~ClassName()`)** is a safety net — it only cleans up unmanaged resources. The GC calls it at an unknown time if the developer forgot to call `Dispose()`. It exists solely because humans forget.

| | `Dispose()` | Finalizer `~` |
|---|---|---|
| **Who calls it** | You (or `using` statement) | The GC |
| **When** | Exactly when you decide | Whenever the GC gets around to it |
| **Cleans up** | Managed + unmanaged | Unmanaged only |
| **Why unmanaged only?** | N/A | Other managed objects might already be finalized — accessing them is unsafe |

## Why the Finalizer Can't Touch Managed Objects

Finalization order is **not guaranteed**. When your finalizer runs, any managed object your class references might already be finalized, collected, or in a corrupt state. Unmanaged resources (raw handles, `IntPtr`) are just numbers — they don't depend on the GC and are always safe to release.

```csharp
~MyClass()
{
    _connection.Dispose();               // DANGEROUS — might already be finalized
    NativeMethods.Close(_nativeHandle);  // SAFE — just a raw handle, always valid
}
```

## The Combined Pattern

You can blend both techniques so that:
- If the developer **remembers** to call `Dispose()` → full cleanup happens immediately, and `GC.SuppressFinalize()` tells the GC to skip the finalizer (no performance penalty).
- If the developer **forgets** → the finalizer catches it and at least releases the unmanaged resources.

The `disposing` parameter is what controls which path is taken:

```csharp
class MyResourceWrapper : IDisposable
{
    private SqlConnection _connection;   // managed
    private IntPtr _nativeHandle;        // unmanaged
    private bool _disposed = false;

    public void Dispose()
    {
        CleanUp(disposing: true);     // full cleanup — managed + unmanaged
        GC.SuppressFinalize(this);    // no need for the safety net anymore
    }

    ~MyResourceWrapper()
    {
        CleanUp(disposing: false);    // safety net — unmanaged only
    }

    private void CleanUp(bool disposing)
    {
        if (_disposed) return;

        if (disposing)
        {
            // ONLY when called from Dispose() — managed objects are alive and safe
            _connection?.Dispose();
        }

        // ALWAYS — unmanaged handles are just numbers, always safe to release
        if (_nativeHandle != IntPtr.Zero)
        {
            NativeMethods.Close(_nativeHandle);
            _nativeHandle = IntPtr.Zero;
        }

        _disposed = true;
    }
}
```

## The Two Paths Visualized

```
Developer calls Dispose()             Developer forgets Dispose()
        │                                      │
        ▼                                      ▼
  CleanUp(true)                        GC finds object unreachable
        │                                      │
        ├─ dispose managed resources           ▼
        ├─ release unmanaged handles     object → f-reachable queue
        ├─ GC.SuppressFinalize()         (survives this GC cycle)
        │                                      │
        ▼                                      ▼
  object collected normally            finalization thread runs ~()
  at next GC (no penalty)                      │
                                               ▼
                                         CleanUp(false)
                                               │
                                               ├─ skip managed (unsafe)
                                               ├─ release unmanaged handles
                                               │
                                               ▼
                                         collected at NEXT GC
                                         (survived an extra cycle)
```

## When You Don't Need a Finalizer

If your class only holds **managed** disposable objects (like `SqlConnection`, `FileStream`, `HttpClient`), you don't need a finalizer at all. Those classes already have their own finalizers as a safety net. Just implement `Dispose()` and clean them up there:

```csharp
class DataService : IDisposable
{
    private SqlConnection _connection;
    private bool _disposed;

    public void Dispose()
    {
        if (_disposed) return;
        _connection?.Dispose();
        _disposed = true;
    }

    // No finalizer — SqlConnection has its own safety net
}
```

Only add a finalizer when your class **directly** holds an unmanaged resource (`IntPtr`, native handle, etc.).
