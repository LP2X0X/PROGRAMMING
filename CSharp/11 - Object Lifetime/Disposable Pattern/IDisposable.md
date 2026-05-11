---
tags:
 - csharp
 - object-lifetime
 - disposable
---

`IDisposable` is how .NET provides **deterministic cleanup** — the ability to release resources at a known, predictable time rather than waiting for the garbage collector. The interface is dead simple:

```csharp
public interface IDisposable
{
    void Dispose();
}
```

One method. No return value. The contract is: after `Dispose()` is called, the object has released its resources and should no longer be used.

---

## Why IDisposable Exists

The GC handles managed memory automatically, but there are resources the GC knows nothing about:
- File handles, network sockets, database connections
- Unmanaged memory (`Marshal.AllocHGlobal`, native allocations)
- OS handles (window handles, registry keys, mutexes)
- GPU resources, hardware device contexts

These are **finite system resources**. If you open a file and wait for the GC to eventually finalize the object, the file stays locked. Other processes (or even other parts of your own code) can't access it until the GC gets around to collecting the object — which could be seconds, minutes, or never if memory pressure is low.

`IDisposable` lets you say "I'm done with this **now**" instead of "I'll let the GC deal with it whenever."

But `IDisposable` is not only for unmanaged resources. It's also used for:
- **Unsubscribing from events** (to prevent memory leaks from event handlers holding references)
- **Cancelling registrations** (e.g., `CancellationTokenRegistration`)
- **Releasing pooled objects** back to a pool
- **Restoring state** (e.g., a timer scope that logs elapsed time on dispose)
- **Flushing buffers** (e.g., `StreamWriter` flushes on dispose)

Any time you need something to happen **at a guaranteed point**, `IDisposable` is the pattern.

---

## Using `IDisposable` — The `using` Statement

The `using` statement is syntactic sugar that guarantees `Dispose()` is called, even if an exception is thrown:

```csharp
using (var stream = new FileStream("data.txt", FileMode.Open))
{
    // use stream
}
// Dispose() called here — guaranteed, even if an exception was thrown
```

The compiler transforms this into:

```csharp
FileStream stream = new FileStream("data.txt", FileMode.Open);
try
{
    // use stream
}
finally
{
    if (stream != null)
        ((IDisposable)stream).Dispose();
}
```

### Using Declaration (C# 8+)

A more concise form that disposes at the end of the enclosing scope:

```csharp
void ProcessFile()
{
    using var stream = new FileStream("data.txt", FileMode.Open);
    using var reader = new StreamReader(stream);

    string content = reader.ReadToEnd();
    // use content

}   // both reader and stream disposed here, in reverse order
```

### Multiple Resources

When you have multiple disposable objects, they're disposed in **reverse order** (last created, first disposed) — like a stack:

```csharp
using var connection = new SqlConnection(connString);
using var command = new SqlCommand(sql, connection);
using var reader = command.ExecuteReader();
// disposed: reader → command → connection
```

---

## Implementing IDisposable — Three Scenarios

### Scenario 1: Class Only Holds Managed Disposable Resources

This is the most common case. Your class wraps other `IDisposable` objects but doesn't directly touch unmanaged resources. **You don't need a finalizer.**

```csharp
public class DataService : IDisposable
{
    private SqlConnection _connection;
    private bool _disposed;

    public DataService(string connectionString)
    {
        _connection = new SqlConnection(connectionString);
        _connection.Open();
    }

    public void Dispose()
    {
        if (_disposed) return;

        _connection?.Dispose();
        _connection = null;

        _disposed = true;
    }
}
```

No finalizer, no `GC.SuppressFinalize`, no `Dispose(bool)` pattern — because there are no unmanaged resources. If the caller forgets to dispose, `SqlConnection` has its own finalizer as a safety net. You don't need to duplicate that.

### Scenario 2: Class Directly Holds Unmanaged Resources

When you directly hold an unmanaged handle, you need the full dispose pattern with a finalizer as a safety net:

```csharp
public class NativeBuffer : IDisposable
{
    private IntPtr _buffer;
    private bool _disposed;

    public NativeBuffer(int size)
    {
        _buffer = Marshal.AllocHGlobal(size);
    }

    public void Dispose()
    {
        Dispose(disposing: true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (_disposed) return;

        if (disposing)
        {
            // Free managed resources here (other IDisposable objects)
        }

        // Free unmanaged resources — always, regardless of disposing flag
        if (_buffer != IntPtr.Zero)
        {
            Marshal.FreeHGlobal(_buffer);
            _buffer = IntPtr.Zero;
        }

        _disposed = true;
    }

    ~NativeBuffer()
    {
        Dispose(disposing: false);
    }
}
```

The `disposing` parameter controls what's safe to touch:

| Called from | `disposing` | Safe to access managed objects? | Why |
|---|---|---|---|
| `Dispose()` (user code) | `true` | Yes — everything is still alive | User called it explicitly, normal execution |
| `~NativeBuffer()` (GC) | `false` | **No** — they might already be finalized | Finalization order is not guaranteed |

### Scenario 3: Derived Class Adds Resources

The `Dispose(bool)` method is `protected virtual` specifically so derived classes can override it:

```csharp
public class EncryptedBuffer : NativeBuffer
{
    private CryptoStream _cryptoStream;

    public EncryptedBuffer(int size) : base(size)
    {
        _cryptoStream = new CryptoStream(/* ... */);
    }

    protected override void Dispose(bool disposing)
    {
        if (disposing)
        {
            _cryptoStream?.Dispose();
            _cryptoStream = null;
        }

        base.Dispose(disposing);  // always call base — it frees the IntPtr
    }

    // No finalizer needed here — base class already has one for the IntPtr
}
```

---

## ObjectDisposedException

After disposal, any method call on the object should throw `ObjectDisposedException`. This is a convention — you implement it manually:

```csharp
public void Write(byte[] data)
{
    ObjectDisposedException.ThrowIf(_disposed, this);  // .NET 7+ helper
    // or manually:
    // if (_disposed) throw new ObjectDisposedException(nameof(NativeBuffer));

    // actual work...
}
```

The CLR does not enforce this automatically. If you skip the check, calling methods on a disposed object may silently corrupt data, throw unrelated exceptions, or appear to work.

---

## IAsyncDisposable (.NET Core 3.0+ / C# 8+)

For resources that require async cleanup (flushing a network stream, closing a database connection with an async protocol), there's `IAsyncDisposable`:

```csharp
public interface IAsyncDisposable
{
    ValueTask DisposeAsync();
}
```

Used with `await using`:

```csharp
await using var connection = new SqlConnection(connString);
await using var reader = await command.ExecuteReaderAsync();
```

A class can implement both `IDisposable` and `IAsyncDisposable`. `await using` calls `DisposeAsync()`. Regular `using` calls `Dispose()`. If you're in an async context, prefer `await using` — some resources can deadlock if their async cleanup is forced through the sync path.

---

## Common Pitfalls

### Forgetting to dispose

The resource stays open until the GC finalizes it (if it even has a finalizer). File locks, connection pool exhaustion, and handle leaks are the symptoms.

### Disposing something you don't own

```csharp
public class BadProcessor : IDisposable
{
    private Stream _stream;

    public BadProcessor(Stream stream)
    {
        _stream = stream;  // caller passed this in — we don't own it
    }

    public void Dispose()
    {
        _stream.Dispose();  // BAD — caller might still need it
    }
}
```

Only dispose resources that your object **created** or **explicitly took ownership of**.

### Using after dispose

```csharp
var stream = new FileStream("data.txt", FileMode.Open);
stream.Dispose();
stream.Read(buffer, 0, 100);  // ObjectDisposedException
```

### Adding a finalizer when you don't need one

If your class only holds managed `IDisposable` objects, **do not add a finalizer**. The objects you hold already have their own finalizers. Adding one to your class just makes it more expensive for the GC (extra generation survival, finalization thread work) for no benefit.

---

## Summary

- `IDisposable` provides **deterministic cleanup** — you control exactly when resources are released.
- Use `using` / `using declaration` to guarantee `Dispose()` is called even during exceptions.
- **No unmanaged resources?** → Simple `Dispose()` method, no finalizer, no `Dispose(bool)` pattern.
- **Direct unmanaged resources?** → Full pattern: `Dispose(bool)` + finalizer + `GC.SuppressFinalize(this)`.
- **Derived classes** override `protected virtual void Dispose(bool)` and call `base.Dispose(disposing)`.
- `IAsyncDisposable` + `await using` for async cleanup scenarios.
- Only dispose what you own. Check `_disposed` before doing work. Never add a finalizer unless you directly hold unmanaged resources.
