---
tags:
 - csharp
 - object-lifetime
 - disposable
 - gc
---

A **finalizable object** is any object whose class defines a finalizer (destructor syntax `~ClassName()`) or in other words managed objects that hold references to unmanaged resources. Finalizers give an object one last chance to clean up unmanaged resources before the GC reclaims its memory. But this safety net comes at a real cost — finalizable objects are significantly more expensive for the GC to manage than regular objects.

```csharp
public class UnmanagedWrapper
{
    private IntPtr _nativeHandle;

    public UnmanagedWrapper()
    {
        _nativeHandle = NativeMethods.CreateResource();
    }

    ~UnmanagedWrapper()  // finalizer — makes this object finalizable
    {
        NativeMethods.ReleaseResource(_nativeHandle);
    }
}
```

```ad-note
The only compelling reason to override Finalize() is if your C# class is using unmanaged resources via PInvoke or complex COM interoperability tasks (typically via various members defined by the System.Runtime.InteropServices.Marshal type). The reason is that under these scenarios you are manipulating memory that the runtime cannot manage.
```

---

## How Finalization Works Internally

The CLR maintains two internal data structures for finalization:

### 1. Finalization Queue

When a finalizable object is allocated, the CLR adds a reference to it on the **finalization queue**. This is not a queue of objects that need finalizing *right now* — it's a registry of every live finalizable object. Simply being on this list means the GC knows the object has a finalizer that may need to run.

### 2. F-Reachable Queue (Freachable Queue)

When the GC runs and determines that a finalizable object is **unreachable** (no root references), it doesn't free the object. Instead, it:

1. Removes the reference from the finalization queue.
2. Moves it to the **f-reachable queue** ("finalizer-reachable").
3. The object is now **reachable again** — the f-reachable queue itself acts as a root. The object **survives** this collection and gets promoted to the next generation.

A dedicated **finalization thread** (a high-priority background thread managed by the CLR) drains the f-reachable queue, calling each object's finalizer one at a time. After the finalizer runs, the reference is removed from the f-reachable queue, and the object is truly unreachable. It will be collected in the **next** GC cycle.

```
Object allocated with finalizer
        │
        ▼
┌─────────────────────┐
│  Finalization Queue  │  ← "I have a finalizer, remember me"
└─────────────────────┘
        │
        │  GC runs, object is unreachable
        ▼
┌─────────────────────┐
│  F-Reachable Queue   │  ← "I need my finalizer called"
└─────────────────────┘     (object is ALIVE again — promoted)
        │
        │  Finalization thread runs ~Destructor()
        ▼
   Object removed from f-reachable queue
   Now truly unreachable
        │
        │  NEXT GC runs
        ▼
   Memory finally reclaimed
```

- Need to check the steps is correct on each GC run with the example below:
 GC #1 runs
    → finds object is unreachable
    → sees it's on the finalization queue
    → moves reference to f-reachable table
    → object SURVIVES (promoted to next generation)
    → GC #1 ends

    ... finalization thread wakes up ...
    ... calls ~Destructor() on the object ...
    ... removes reference from f-reachable table ...
    ... object is now truly unreachable ...

  GC #2 runs (sometime later)
    → finds the object is unreachable (no more f-reachable reference)
    → no finalizer registered anymore
    → frees the managed memory

---

## Why Finalizable Objects Are Expensive

### 1. They survive at least one extra GC cycle

A normal unreachable object is collected immediately. A finalizable object survives the collection that discovers it's unreachable, gets promoted to a higher generation, and isn't freed until a subsequent collection. This means:
- An object that would have died in Gen 0 now lives into Gen 1 (or Gen 2).
- Reclaiming it requires a more expensive higher-generation collection.

### 2. They promote everything they reference

When a finalizable object survives, **every object it references also survives** (the GC must keep the entire object graph alive because the finalizer might access any of it). One finalizable object can keep an entire tree of objects alive longer than necessary.

```csharp
~MyClass()
{
    // The GC can't know whether the finalizer will touch _data,
    // so it must keep _data alive until after this runs
    Console.WriteLine(_data.Length);
}
private byte[] _data = new byte[10_000_000];  // 10 MB kept alive unnecessarily
```

### 3. Finalization order is not guaranteed

The CLR makes **no guarantee** about the order in which finalizers run. If object A references object B and both are finalizable, B's finalizer might run before A's. This means inside a finalizer, you **cannot safely access other managed finalizable objects** — they might already be finalized.

This is the reason the Dispose pattern passes `false` to `CleanUp(disposing)` when called from the finalizer — you should only clean up **unmanaged** resources, not touch managed objects.

### 4. Finalizers run on a single thread

All finalizers execute sequentially on one dedicated finalization thread. A slow or blocking finalizer delays finalization of every other object in the queue. A finalizer that **deadlocks or hangs** will stop all finalization permanently — those objects and everything they reference will leak for the life of the process.

### 5. Finalizers are not guaranteed to run at all

In extreme cases (process crash, `Environment.FailFast()`, unhandled exception in another finalizer, or the app domain being unloaded aggressively), finalizers may be skipped entirely. They are a **best-effort** safety net, not a guarantee.

---

## `GC.SuppressFinalize()` — Opting Out

If an object's resources are already cleaned up (because `Dispose()` was called), there's no reason to run the finalizer. `GC.SuppressFinalize(this)` removes the object from the finalization queue, so:

- It won't be moved to the f-reachable queue.
- It won't survive an extra GC cycle.
- It will be collected immediately like a normal object.

```csharp
public void Dispose()
{
    NativeMethods.ReleaseResource(_nativeHandle);
    _nativeHandle = IntPtr.Zero;

    GC.SuppressFinalize(this);  // finalizer is no longer needed
}
```

This is why the Dispose pattern always calls `GC.SuppressFinalize(this)` — it avoids the entire finalization penalty when the user properly disposes the object.

---

## `GC.ReRegisterForFinalize()` — Resurrection

You can re-register an object for finalization after it's been suppressed (or even after its finalizer has already run):

```csharp
GC.ReRegisterForFinalize(this);
```

This is an advanced and rare technique called **object resurrection** — the finalizer makes the object reachable again (e.g., by assigning `this` to a static field), and re-registers it for finalization so the finalizer will run again next time the object becomes unreachable. In practice, this is almost never a good idea.

---

## Practical Guidelines

**Avoid finalizers unless you directly hold unmanaged resources.** If your class only holds managed objects (even ones like `SqlConnection` that wrap unmanaged resources), you don't need a finalizer — just implement `IDisposable` and dispose the managed objects. They have their own finalizers as a fallback.

**Always pair a finalizer with `IDisposable`.** The finalizer is the safety net; `Dispose()` is the normal path. Call `GC.SuppressFinalize(this)` in `Dispose()` so well-behaved callers don't pay the finalization cost.

**Keep finalizers fast and simple.** Don't block, don't lock, don't throw, don't access other finalizable managed objects. Release your unmanaged handle and get out.

**Prefer `SafeHandle` over raw finalizers.** The `SafeHandle` class (and its subclasses like `SafeFileHandle`) encapsulates the release of an unmanaged handle with proper reference counting and critical finalization. It's more robust than writing your own finalizer:

```csharp
public class MyResource : SafeHandleZeroOrMinusOneIsInvalid
{
    public MyResource() : base(ownsHandle: true)
    {
        SetHandle(NativeMethods.CreateResource());
    }

    protected override bool ReleaseHandle()
    {
        return NativeMethods.CloseResource(handle);
    }
}
```

`SafeHandle` uses **critical finalization** (`CriticalFinalizerObject`), which guarantees its finalizer runs even in constrained execution regions and after normal finalizers — making it significantly more reliable than a plain `~Destructor()`.
