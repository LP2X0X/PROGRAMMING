---
tags:
 - csharp
 - object-lifetime
 - gc
---

The .NET garbage collector doesn't scan the entire managed heap every time it runs — that would be far too slow for any real application. Instead, it uses a **generational model** based on a simple empirical observation: most objects die young. A local variable inside a method, a temporary string from concatenation, a short-lived `Task` — these are created and abandoned constantly. Meanwhile, objects like your application's main window or a singleton service live for the entire process. Treating these the same would be wasteful.

The generational design lets the GC collect cheap, short-lived garbage frequently and only inspect long-lived objects when it has to.

---

## The Three Generations

Every object on the managed heap belongs to one of three generations:

### Generation 0 — Nursery

- Where **all newly allocated objects** start (with one exception — see Large Object Heap below).
- This is small by design — typically around **256 KB to a few MB** — so it fits in the CPU's L2/L3 cache, making scanning fast.
- Gen 0 collections happen **most frequently** and are the fastest.
- Most objects die here. If your method allocates a `List<int>`, uses it, and returns, that list never leaves Gen 0.

### Generation 1 — Buffer Zone

- Objects that **survived one Gen 0 collection** get promoted here.
- Acts as a **buffer between short-lived and long-lived objects**. It gives objects a second chance — maybe they were allocated just before a Gen 0 collection and appear to be long-lived, but they're actually about to become unreachable.
- Gen 1 is also small and relatively cheap to collect.
- Collected less frequently than Gen 0, but more frequently than Gen 2.

### Generation 2 — Long-Lived / Tenured

- Objects that **survived a Gen 1 collection** get promoted here.
- This is where long-lived objects end up: singletons, caches, static data structures, the main application objects.
- Gen 2 is the **largest** generation and the **most expensive** to collect — a Gen 2 collection (also called a **full collection**) walks the entire managed heap.
- Once an object is in Gen 2, it stays Gen 2 forever. There is no Gen 3.

```
Allocation → Gen 0 → (survives) → Gen 1 → (survives) → Gen 2
                ↓                    ↓                    ↓
            collected            collected            collected
            (most die here)
```

---

## How a Collection Works

When the GC decides to collect, it targets a specific generation. A key rule: **collecting generation N always collects all lower generations too.**

- **Gen 0 collection**: Scans only Gen 0. Fast, frequent. Survivors promote to Gen 1.
- **Gen 1 collection**: Scans Gen 0 and Gen 1. Survivors of Gen 1 promote to Gen 2.
- **Gen 2 collection (full GC)**: Scans Gen 0, Gen 1, and Gen 2. The most expensive. The GC tries hard to avoid these.

The process:

1. **Suspension** — managed threads are paused (this is the "stop the world" pause, though background GC minimizes this — see below).
2. **Mark** — starting from GC roots (stack variables, static fields, GC handles), the collector walks all references and marks reachable objects as alive.
3. **Sweep/Compact** — unmarked objects are reclaimed. Surviving objects may be **compacted** (moved together to eliminate fragmentation), and references are updated to point to the new locations. Survivors are promoted to the next generation.
4. **Resume** — threads resume execution.

---

## What Triggers a Collection

The GC doesn't run on a fixed timer. It runs when:

| Trigger | What happens |
|---------|-------------|
| **Gen 0 budget exceeded** | When enough allocations have filled Gen 0's budget (a dynamically tuned threshold), a Gen 0 collection fires. This is the most common trigger. |
| **Gen 1 / Gen 2 budget exceeded** | If promoted objects push Gen 1 or Gen 2 past their thresholds, a higher-generation collection runs. |
| **System low on physical memory** | The OS signals memory pressure, and the GC responds with a more aggressive collection. |
| **`GC.Collect()` called explicitly** | Forces a collection. Almost always a bad idea in production — it defeats the GC's tuning heuristics. |

The GC dynamically adjusts generation budgets based on your application's allocation patterns. If Gen 0 objects tend to survive, it may shrink the Gen 0 budget so collections happen sooner (promoting fewer objects at a time). If most Gen 0 objects die, it may grow the budget to reduce collection frequency.

---

## The Large Object Heap (LOH) and Pinned Object Heap (POH)

### Large Object Heap (LOH)

Objects **85,000 bytes or larger** are not allocated in the normal generational heap. They go directly to the **Large Object Heap**, which is logically part of Gen 2.

- LOH objects are **not compacted by default** — moving a 10 MB array is expensive and would cause a long pause. This means the LOH can become fragmented over time.
- Since .NET 4.5.1, you can opt in to LOH compaction: `GCSettings.LargeObjectHeapCompactionMode = GCLargeObjectHeapCompactionMode.CompactOnce;`
- LOH is only collected during **Gen 2 (full) collections**.
- Common LOH residents: large arrays, big strings, large `byte[]` buffers.

### Pinned Object Heap (POH) — .NET 5+

Objects that need to be **pinned** (prevented from moving during compaction, typically for interop with native code) go to the **Pinned Object Heap**. This avoids the fragmentation problems that pinned objects used to cause on the normal heap.

```csharp
// Allocated on the POH — won't move, won't fragment the normal heap
byte[] buffer = GC.AllocateArray<byte>(4096, pinned: true);
```

---

## Background vs Workstation vs Server GC

The GC has different modes that affect how and when generations are collected:

### Workstation GC (default for client apps)
- GC runs on the thread that triggered the allocation.
- One managed heap for the whole process.
- Lower throughput, but lower latency — suitable for UI apps.

### Server GC (default for ASP.NET / server apps)
- Creates **one heap and one dedicated GC thread per logical CPU core**.
- Collections happen in parallel across all heaps.
- Higher throughput, but higher memory usage.

Enable in your project file:
```xml
<PropertyGroup>
    <ServerGarbageCollection>true</ServerGarbageCollection>
</PropertyGroup>
```

### Background GC (enabled by default in both modes)

- Gen 0 and Gen 1 collections are always **foreground** (blocking / stop-the-world), but they're fast because these generations are small.
- Gen 2 collections run on a **background thread** concurrently with your application. The mark phase for Gen 2 happens mostly without stopping your threads, reducing pause times significantly.
- Without background GC (concurrent GC disabled), a full Gen 2 collection would freeze all managed threads for the entire duration — unacceptable for interactive or latency-sensitive applications.

---

## Inspecting Generations in Code

The `System.GC` class exposes generation information:

```csharp
var obj = new object();

// Which generation is this object in?
Console.WriteLine(GC.GetGeneration(obj));  // 0

GC.Collect(0);  // force Gen 0 collection
Console.WriteLine(GC.GetGeneration(obj));  // 1 (promoted)

GC.Collect(1);  // force Gen 1 collection
Console.WriteLine(GC.GetGeneration(obj));  // 2 (promoted)

GC.Collect(2);  // full collection
Console.WriteLine(GC.GetGeneration(obj));  // 2 (stays — no Gen 3)
```

```csharp
// How many collections have occurred for each generation?
Console.WriteLine($"Gen 0 collections: {GC.CollectionCount(0)}");
Console.WriteLine($"Gen 1 collections: {GC.CollectionCount(1)}");
Console.WriteLine($"Gen 2 collections: {GC.CollectionCount(2)}");

// In a healthy app, you'll see something like:
// Gen 0 collections: 542
// Gen 1 collections: 48
// Gen 2 collections: 3
// The ratio tells you the GC is doing its job — most work in Gen 0.
```

---

## Practical Implications

**Allocate and release quickly.** Objects that die in Gen 0 are nearly free. The GC is heavily optimized for this pattern. The expensive case is objects that live *just long enough* to get promoted to Gen 1 or Gen 2 and then die — they survive cheap collections and force expensive ones.

**Avoid mid-life crisis objects.** An object that lives through a few Gen 0 collections (because it's referenced by an ongoing async operation, an event handler, or a cache with slow expiry) gets promoted to Gen 1 or Gen 2 unnecessarily. These "mid-life" objects are the most expensive for the GC because they require a higher-generation collection to reclaim.

**Watch your Gen 2 collection count.** If Gen 2 collections are frequent, your app is promoting too many objects or holding too many large/long-lived references. Use a profiler (dotnet-counters, PerfView, Visual Studio Diagnostic Tools) to investigate.

**Don't call `GC.Collect()` in production.** It forces a full blocking collection, promotes surviving objects unnecessarily, and disrupts the GC's self-tuning. The GC almost certainly knows better than you when to collect. The rare exceptions: after a massive one-time operation that you know released a lot of memory, or during a natural loading screen / downtime in a game.

---

## Summary

- Objects start in **Gen 0** (small, fast, collected often). Survivors promote to **Gen 1** (buffer), then **Gen 2** (long-lived, expensive to collect).
- Collecting Gen N also collects all lower generations.
- **Large objects (>= 85 KB)** go to the LOH, which is logically Gen 2 and not compacted by default.
- The GC self-tunes generation budgets based on your allocation patterns.
- **Background GC** runs Gen 2 collections concurrently to minimize pause times.
- **Server GC** uses one heap per core for throughput; **Workstation GC** uses one heap for lower latency.
- Most objects should die in Gen 0. Watch for mid-life promotions and frequent Gen 2 collections as signs of GC pressure.
