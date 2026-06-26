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

The GC **does not guarantee finalization order**. When the GC discovers that multiple objects are unreachable, it puts all of them on the finalization queue and processes them in **arbitrary order**. Your finalizer has no way to know whether the managed objects it references have already been finalized or not.

### The Problem — Step by Step

```csharp
class MyClass : IDisposable
{
    private FileStream _stream;     // managed object (wraps an OS file handle)
    private IntPtr _nativeHandle;   // unmanaged resource (just a number)

    ~MyClass()
    {
        _stream.Dispose();                   // DANGEROUS
        NativeMethods.Close(_nativeHandle);  // SAFE
    }
}
```

Here's what can happen at runtime:

```
1. GC finds both MyClass and FileStream are unreachable
2. GC puts BOTH on the finalization queue
3. GC finalizes FileStream FIRST (random order)
   → FileStream's finalizer closes its internal OS handle
   → FileStream's internal state is now invalid (handle = 0, buffer = null)
4. GC finalizes MyClass
   → _stream.Dispose() — calling a method on a zombie object
   → ObjectDisposedException or NullReferenceException or corrupted state
```

### Why Unmanaged Resources Are Safe

The GC is in charge of **three things**: managed memory, managed objects, and **finalization scheduling**. It discovers unreachable finalizable objects, puts them on the finalization queue, and runs the finalizer thread. But there's a critical gap — the GC doesn't know **what** an unmanaged resource is or **how** to release it. An `IntPtr` is just a number to the GC. It can't finalize a file handle. It can't close a socket. Your finalizer code is the only thing that knows how.

That's exactly why unmanaged resources are safe to touch in a finalizer — the GC never touches them, so they're always in the same state you left them:

```
GC controls:                         GC does NOT control:
──────────────────────               ───────────────────────────
FileStream object (memory)           The OS file handle inside it
FileStream finalization (scheduling) How to release that handle
SqlConnection object (memory)        The native socket inside it
```

The GC can finalize the **managed wrapper** (the `FileStream` object) in any order. But the **raw handle** (`IntPtr`) inside it is just a number sitting in memory — the GC doesn't destroy it, move it, or zero it out. It stays valid until **your code** calls the OS to release it.

### The Rule

- **Managed objects** → the GC controls their lifetime AND finalization order → another object's finalizer may have already corrupted them → **don't touch** in your finalizer
- **Unmanaged resources** → the GC controls **nothing** about them → they stay exactly as you left them → the finalizer is your **last chance** to release them

```csharp
~MyClass()
{
    // _stream?.Dispose();              // NO — managed, might be dead
    NativeMethods.Close(_nativeHandle); // YES — unmanaged, always valid
}
```

```ad-note
"Cleaning managed resources" in `Dispose()` means calling `.Dispose()` on other `IDisposable` objects you hold. It's a chain: your `Dispose()` calls their `Dispose()`, which eventually reaches the object that holds the raw handle and releases it. The finalizer skips this chain because the intermediate objects may be dead.
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
