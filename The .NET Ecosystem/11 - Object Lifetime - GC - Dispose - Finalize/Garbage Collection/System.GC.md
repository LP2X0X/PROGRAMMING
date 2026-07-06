---
tags:
 - csharp
 - object-lifetime
 - gc
---

## What Is System.GC?

`System.GC` is a static class in `mscorlib` / `System.Runtime` that lets you **programmatically interact with the garbage collector**. It exposes methods to force collections, query generation info, manage memory pressure, and control finalization behavior.

```ad-warning
You will rarely need this class in application code. The GC's self-tuning heuristics almost always outperform manual intervention. The primary use case is when you are writing classes that manage **unmanaged resources** or building infrastructure/framework-level code (memory pools, custom allocators, diagnostic tools).
```

---

## Core Methods

### GC.Collect() — Forcing a Collection

Forces an immediate garbage collection. Multiple overloads give you control over scope:

```csharp
// Force a FULL collection (Gen 0 + Gen 1 + Gen 2)
GC.Collect();

// Collect only up to the specified generation
GC.Collect(0);   // Gen 0 only
GC.Collect(1);   // Gen 0 + Gen 1
GC.Collect(2);   // Gen 0 + Gen 1 + Gen 2 (same as parameterless)

// With collection mode — controls blocking behavior
GC.Collect(2, GCCollectionMode.Forced);      // force it now, blocking
GC.Collect(2, GCCollectionMode.Optimized);   // let the GC decide if now is a good time
GC.Collect(2, GCCollectionMode.Default);     // same as Forced

// With blocking parameter (.NET Core / .NET 5+)
GC.Collect(2, GCCollectionMode.Forced, blocking: true);   // stop-the-world
GC.Collect(2, GCCollectionMode.Forced, blocking: false);  // background (non-blocking)

// With compacting parameter
GC.Collect(2, GCCollectionMode.Forced, blocking: true, compacting: true);
```

| Parameter | Purpose |
|---|---|
| `generation` | Maximum generation to collect (0, 1, or 2) |
| `mode` | `Forced` = do it now, `Optimized` = GC may skip if unnecessary |
| `blocking` | `true` = blocks all threads until done, `false` = runs concurrently |
| `compacting` | `true` = compact the heap after collection (reduces fragmentation) |

```ad-warning
**Why you almost never call GC.Collect():**
1. It forces a **full blocking collection** by default — all managed threads freeze.
2. It **promotes surviving objects** to the next generation, making future collections more expensive.
3. It **defeats the GC's self-tuning** — the GC dynamically adjusts generation budgets based on your allocation patterns. A manual collect resets that learning.
4. It gives a false sense of control — the GC already knows when to collect.
```

**The rare exceptions where `GC.Collect()` is acceptable:**

```csharp
// 1. After a massive one-time operation that released lots of memory
void LoadLevel(string levelName)
{
    UnloadCurrentLevel();   // drops thousands of objects
    GC.Collect();           // reclaim now — we're in a loading screen anyway
    GC.WaitForPendingFinalizers();
    GC.Collect();           // collect objects freed by finalizers
    LoadNewLevel(levelName);
}

// 2. In benchmarking — ensure a clean baseline
GC.Collect();
GC.WaitForPendingFinalizers();
GC.Collect();
var sw = Stopwatch.StartNew();
// ... code under test ...

// 3. Diagnostic / profiling code
```

---

### GC.WaitForPendingFinalizers()

Blocks the calling thread until **all objects in the finalization queue have been finalized** (their `~Destructor()` has run).

```csharp
GC.Collect();
GC.WaitForPendingFinalizers();  // wait for finalizer thread to finish
GC.Collect();                   // now collect the finalized objects
```

**Why the double-collect pattern?**

```
Step 1: GC.Collect()
  → Finds unreachable objects WITH finalizers
  → Moves them to the finalization queue (f-reachable queue)
  → They are NOT collected yet — they're kept alive for finalization

Step 2: GC.WaitForPendingFinalizers()
  → The dedicated finalizer thread runs each object's ~Destructor()
  → After finalization, objects become truly unreachable

Step 3: GC.Collect()  (second call)
  → NOW those finalized objects are collected and their memory is freed
```

Without the second `GC.Collect()`, finalized objects sit in memory until the **next natural collection** — which could be a while.

See also: [[Finalizable Objects]]

---

### GC.SuppressFinalize(object obj)

Tells the GC: "Don't bother running the finalizer for this object — I've already cleaned up manually."

```csharp
public class MyResource : IDisposable
{
    private IntPtr _handle;

    public void Dispose()
    {
        CloseHandle(_handle);
        GC.SuppressFinalize(this);  // finalizer is no longer needed
    }

    ~MyResource()
    {
        CloseHandle(_handle);  // safety net — only runs if Dispose() wasn't called
    }
}
```

**Why this matters for performance:**

- Objects with finalizers are **more expensive** — the GC puts them on the finalization queue instead of collecting them immediately. They survive at least one extra collection cycle.
- Calling `SuppressFinalize` removes the object from the finalization queue, so it can be collected in the **same** GC cycle like a normal object.
- This is why every correct `Dispose()` implementation calls `GC.SuppressFinalize(this)` at the end.

```
Without SuppressFinalize:
  Unreachable → finalization queue → finalizer runs → NEXT collection frees memory
  (survives at least 2 collections)

With SuppressFinalize (after Dispose):
  Unreachable → collected immediately
  (survives 1 collection like normal objects)
```

See also: [[Finalizer and Dispose]], [[IDisposable]]

---

### GC.ReRegisterForFinalize(object obj)

The opposite of `SuppressFinalize` — puts the object **back** on the finalization queue. Extremely rare.

```csharp
public void Resurrect()
{
    GC.ReRegisterForFinalize(this);
    // Object will be finalized again when it becomes unreachable
}
```

The main use case is **object resurrection** — when a finalizer saves a reference to `this` somewhere (e.g., a static list), making the object reachable again. You'd call `ReRegisterForFinalize` so the finalizer runs again when the object truly becomes unreachable the second time.

```ad-warning
Object resurrection is almost always a design smell. If you find yourself using `ReRegisterForFinalize`, reconsider your architecture.
```

---

## Query Methods

### GC.GetGeneration(object obj)

Returns which [[Object Generations|generation]] (0, 1, or 2) an object currently belongs to.

```csharp
var obj = new object();
Console.WriteLine(GC.GetGeneration(obj));  // 0 — just allocated

GC.Collect(0);
Console.WriteLine(GC.GetGeneration(obj));  // 1 — promoted

GC.Collect(1);
Console.WriteLine(GC.GetGeneration(obj));  // 2 — promoted again

GC.Collect(2);
Console.WriteLine(GC.GetGeneration(obj));  // 2 — stays (no Gen 3)
```

Useful for diagnostics — if objects you expect to die young are reaching Gen 2, you have a memory issue.

---

### GC.CollectionCount(int generation)

Returns how many times a specific generation has been collected since the process started.

```csharp
Console.WriteLine($"Gen 0: {GC.CollectionCount(0)}");  // e.g., 542
Console.WriteLine($"Gen 1: {GC.CollectionCount(1)}");  // e.g., 48
Console.WriteLine($"Gen 2: {GC.CollectionCount(2)}");  // e.g., 3
```

**Healthy ratio:** Gen 0 >> Gen 1 >> Gen 2. If Gen 2 collections are frequent, your app is promoting too many objects — investigate with a profiler.

---

### GC.MaxGeneration

Returns the highest supported generation number (currently always **2**).

```csharp
Console.WriteLine(GC.MaxGeneration);  // 2
```

---

### GC.GetTotalMemory(bool forceFullCollection)

Returns the estimated number of bytes currently allocated on the managed heap.

```csharp
// Approximate — fast, no collection triggered
long approx = GC.GetTotalMemory(forceFullCollection: false);

// More accurate — forces a full GC first, then measures
long accurate = GC.GetTotalMemory(forceFullCollection: true);

Console.WriteLine($"Managed heap: {accurate / 1024.0 / 1024.0:F2} MB");
```

---

### GC.GetGCMemoryInfo() — .NET Core 3.0+

Returns detailed memory statistics in a `GCMemoryInfo` struct. Much more information than `GetTotalMemory`:

```csharp
GCMemoryInfo info = GC.GetGCMemoryInfo();

Console.WriteLine($"Heap size:       {info.HeapSizeBytes / 1024 / 1024} MB");
Console.WriteLine($"Committed:       {info.TotalCommittedBytes / 1024 / 1024} MB");
Console.WriteLine($"Promoted:        {info.PromotedBytes / 1024 / 1024} MB");
Console.WriteLine($"Pinned objects:  {info.PinnedObjectsCount}");
Console.WriteLine($"Pause time:      {info.PauseTimePercentage}%");
Console.WriteLine($"Concurrent:      {info.Concurrent}");
Console.WriteLine($"Compacted:       {info.Compacted}");
```

---

### GC.GetTotalAllocatedBytes(bool precise) — .NET Core 3.0+

Returns the **total bytes allocated** since process start (not current heap size — this number only goes up). Useful for measuring allocation rate.

```csharp
long before = GC.GetTotalAllocatedBytes(precise: true);
// ... do some work ...
long after = GC.GetTotalAllocatedBytes(precise: true);

Console.WriteLine($"Allocated {after - before} bytes during the operation");
```

---

## Memory Pressure Methods

### GC.AddMemoryPressure / GC.RemoveMemoryPressure

The GC only sees managed heap allocations. If your object holds a large unmanaged allocation (e.g., a native bitmap, a large `Marshal.AllocHGlobal` block), the GC doesn't know about it. From the GC's perspective, your object is a tiny managed reference — it has no urgency to collect it, even though freeing it would release megabytes of native memory.

These methods let you **tell the GC** about unmanaged memory your object holds:

```csharp
class NativeBitmap : IDisposable
{
    private IntPtr _nativeBuffer;
    private readonly long _size;

    public NativeBitmap(int width, int height)
    {
        _size = width * height * 4L;  // RGBA, 4 bytes per pixel
        _nativeBuffer = Marshal.AllocHGlobal((int)_size);

        // Tell the GC: "I'm holding this much unmanaged memory"
        GC.AddMemoryPressure(_size);
    }

    public void Dispose()
    {
        Marshal.FreeHGlobal(_nativeBuffer);

        // Tell the GC: "I released that unmanaged memory"
        GC.RemoveMemoryPressure(_size);

        GC.SuppressFinalize(this);
    }

    ~NativeBitmap()
    {
        Marshal.FreeHGlobal(_nativeBuffer);
        GC.RemoveMemoryPressure(_size);
    }
}
```

**Without** `AddMemoryPressure`: You allocate 100 `NativeBitmap` objects, each holding 10 MB of native memory (1 GB total), but the managed heap only sees 100 tiny references (~2 KB total). The GC has no reason to collect aggressively.

**With** `AddMemoryPressure`: The GC factors the reported pressure into its scheduling decisions. It may trigger collections sooner, which runs finalizers or allows Dispose to be called, freeing the native memory before the system runs out.

```ad-note
`AddMemoryPressure` and `RemoveMemoryPressure` must always be **balanced** — every Add must have a corresponding Remove. An imbalanced pair will cause the GC to over-collect (too much pressure) or under-collect (released pressure never reported).
```

---

## Allocation Methods

### GC.AllocateArray<T> and GC.AllocateUninitializedArray<T> — .NET 5+

Allocate arrays with special options:

```csharp
// Allocate on the Pinned Object Heap (POH) — won't move during compaction
byte[] pinnedBuffer = GC.AllocateArray<byte>(4096, pinned: true);

// Allocate without zeroing memory — faster for buffers you'll overwrite immediately
byte[] rawBuffer = GC.AllocateUninitializedArray<byte>(4096);
```

**Pinned arrays** are useful for interop scenarios where native code needs a stable pointer. Before the POH existed, you had to use `GCHandle.Alloc(arr, GCHandleType.Pinned)` which caused heap fragmentation.

**Uninitialized arrays** skip the default zero-fill. Only use when you're going to write every byte before reading — otherwise you'll read garbage values.

See also: [[Large Object Heap (LOH)]]

---

## Summary — When to Use Each Method

| Method | When to use |
|---|---|
| `GC.Collect()` | Loading screens, benchmarks, diagnostics. **Never** in normal application flow. |
| `GC.WaitForPendingFinalizers()` | After `GC.Collect()` when you need finalizers to complete before proceeding. |
| `GC.SuppressFinalize(this)` | **Always** at the end of `Dispose()` when your class has a finalizer. |
| `GC.ReRegisterForFinalize(obj)` | Almost never — only for object resurrection patterns. |
| `GC.GetGeneration(obj)` | Diagnostics — checking if objects are promoted unexpectedly. |
| `GC.CollectionCount(gen)` | Monitoring GC health — watch the ratio between generations. |
| `GC.GetTotalMemory(bool)` | Quick check of managed heap size. |
| `GC.GetGCMemoryInfo()` | Detailed memory diagnostics (.NET Core 3.0+). |
| `GC.GetTotalAllocatedBytes(bool)` | Measuring allocation rate between two points. |
| `GC.AddMemoryPressure(long)` | When your object holds large unmanaged allocations the GC doesn't know about. |
| `GC.RemoveMemoryPressure(long)` | When you release those unmanaged allocations. |
| `GC.AllocateArray<T>(pinned)` | Interop buffers that must not move (.NET 5+). |
| `GC.AllocateUninitializedArray<T>()` | Performance-critical buffers you'll overwrite immediately (.NET 5+). |
