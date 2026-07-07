---
tags:
 - csharp
 - object-lifetime
 - gc
---

The .NET garbage collector (GC) is an automatic memory manager that handles allocation and deallocation of objects on the managed heap. It eliminates entire classes of bugs — dangling pointers, double frees, most memory leaks — by taking ownership of object lifetimes. But "automatic" does not mean "free." Understanding how the GC works is essential for writing performant .NET code.

---

## The Managed Heap

When a .NET process starts, the CLR reserves a contiguous region of virtual memory called the **managed heap**. All reference-type objects are allocated here. The heap is divided into distinct regions:

| Heap | What goes there | Compacted? |
|---|---|---|
| **SOH** (Small Object Heap) | Objects < 85,000 bytes | Yes — surviving objects slide together |
| **LOH** (Large Object Heap) | Objects >= 85,000 bytes | No by default (opt-in since .NET 4.5.1) |
| **POH** (Pinned Object Heap, .NET 5+) | Objects pinned for native interop | No — native code holds raw pointers |

See [[Determining if an Object is Live]] for how the GC decides which objects on these heaps are still in use.

---

## Generational Model

The GC uses a **generational** strategy based on the observation that most objects die young. Objects are grouped into three generations:

- **Gen 0** — where all new objects start. Small, collected frequently, very fast.
- **Gen 1** — buffer zone for objects that survived one Gen 0 collection.
- **Gen 2** — long-lived / tenured objects. Largest generation, most expensive to collect.

Collecting generation N **always** collects all lower generations. So a Gen 2 collection scans Gen 0, Gen 1, and Gen 2.

Objects >= 85,000 bytes skip the generational system and go directly to the **LOH**, which is logically part of Gen 2. See [[Large Object Heap (LOH)]] for details.

For a full breakdown of how generations work, promotion, budgets, and triggers, see [[Object Generations]].

---

## The Three Phases of Collection

When the GC decides to collect, it executes three phases:

### Phase 1: Mark

Starting from **GC roots** (stack variables, static fields, GC handles, CPU registers), the collector traces every reference chain and marks all reachable objects as **live**. Anything not marked is garbage.

This is a **tracing** algorithm, not reference counting — circular references are handled naturally because the GC only cares about reachability from roots, not reference counts.

See [[Determining if an Object is Live]] for the full details on roots, the object graph, and how tracing works.

### Phase 2: Sweep

The GC identifies all unmarked (dead) objects and reclaims their memory. On the SOH, this creates gaps in the heap where dead objects used to be.

### Phase 3: Compact

After sweeping, the SOH has holes scattered throughout. The GC slides surviving objects together to eliminate these gaps, producing a contiguous block of free memory at the end. All references to moved objects are updated to point to their new locations.

Compaction enables **bump-pointer allocation** — allocating a new object is just incrementing a pointer, which is extremely fast (comparable to stack allocation).

See [[Compaction Process]] for the full details on how compaction works, its costs, and the optimization strategies the runtime uses.

```
Before compaction:
[ObjA][  dead  ][ObjB][  dead  ][ObjC]   <- fragmented

After compaction:
[ObjA][ObjB][ObjC][      free space     ] <- contiguous
```

---

## What Triggers a Collection

The GC does not run on a fixed timer. Collections are triggered by:

| Trigger | Description |
|---|---|
| **Gen 0 budget exceeded** | Enough allocations filled Gen 0's threshold (dynamically tuned). Most common trigger. |
| **Gen 1 / Gen 2 budget exceeded** | Promoted objects push higher generations past their thresholds. |
| **System memory pressure** | The OS signals low physical memory. GC responds with aggressive collection. |
| **`GC.Collect()` called** | Forces a collection. Almost always a bad idea in production — defeats the GC's self-tuning. See [[System.GC]]. |

The GC **dynamically adjusts** generation budgets based on your application's allocation patterns. If most Gen 0 objects die quickly, it grows the Gen 0 budget to collect less often. If many survive, it shrinks the budget to promote fewer at a time.

---

## Thread Suspension and GC Modes

During collection, managed threads must be paused at **safe points** so the GC can safely inspect and move objects. This is the "stop the world" pause.

### Workstation GC (default for client/desktop apps)
- Single managed heap for the whole process.
- GC runs on the thread that triggered the allocation.
- Lower throughput, but lower latency — suitable for UI applications.

### Server GC (default for ASP.NET / server apps)
- Creates **one heap and one dedicated GC thread per logical CPU core**.
- Collections happen in parallel across all heaps.
- Higher throughput, but higher memory usage.

```xml
<PropertyGroup>
    <ServerGarbageCollection>true</ServerGarbageCollection>
</PropertyGroup>
```

### Background GC (enabled by default in both modes)
- **Gen 0 and Gen 1** collections are always **foreground** (blocking), but fast because these generations are small.
- **Gen 2** collections run on a **background thread** concurrently with your application. The mark phase happens mostly without stopping your threads.
- Without background GC, a full Gen 2 collection would freeze all managed threads for its entire duration.

Gen 0 and Gen 1 live in a dedicated **ephemeral segment** — a contiguous memory block designed to fit in CPU cache for fast scanning. See [[Ephemeral Generations and Segments]] for how this works.

---

## Finalization and Its Impact on GC

Objects with a finalizer (`~ClassName()`) are treated specially by the GC:

1. They are registered on the **finalization queue** at allocation time.
2. When found unreachable, they are **not collected** — instead they're moved to the **f-reachable queue**, which makes them reachable again.
3. They **survive** the current collection and get **promoted** to the next generation.
4. A dedicated **finalization thread** runs their finalizer.
5. Only on the **next** GC pass (after finalization) can their memory be reclaimed.

This means finalizable objects require **at least two GC passes** to collect, and they drag their entire referenced object graph along for the ride.

See [[Finalizable Objects]] for the full explanation of finalization queues, the cascade promotion effect, and why `GC.SuppressFinalize(this)` matters for performance.

---

## The GC as a Memory Manager — Not a Lifecycle Manager

The GC reclaims **memory** — the bytes on the managed heap. It does **not**:

- Call `Dispose()`
- Unsubscribe from events
- Return pooled connections
- Stop timers or background work
- Flush buffers

These are **lifecycle concerns** handled by the [[IDisposable]] pattern. Relying on the GC to "clean up" resources beyond memory is a common source of logical leaks.

See [[What Happens If You Forget to Call Dispose on Managed Resources]] for the real-world consequences.

---

## Practical Guidelines

**Allocate and release quickly.** Objects that die in Gen 0 are nearly free. The GC is heavily optimized for this "infant mortality" pattern.

**Avoid mid-life crisis objects.** Objects that live just long enough to get promoted to Gen 1 or Gen 2, then die, are the most expensive — they survive cheap collections and force expensive ones.

**Watch your Gen 2 collection count.** Frequent Gen 2 collections indicate too many promotions or too many long-lived references. Use profiling tools (dotnet-counters, PerfView, Visual Studio Diagnostic Tools) to investigate.

**Don't call `GC.Collect()` in production.** It forces a full blocking collection, promotes surviving objects unnecessarily, and disrupts the GC's self-tuning. See [[System.GC]] for the rare exceptions.

**Avoid unnecessary finalizers.** If your class only holds managed `IDisposable` objects, you don't need a finalizer — just implement `IDisposable`. The objects you hold already have their own finalizers as a safety net. Adding one to your class just increases GC pressure. See [[Finalizable Objects]].

---

## Summary

| Concept | Key Point |
|---|---|
| **Heap structure** | SOH (compacted), LOH (not compacted), POH (pinned, .NET 5+) |
| **Generations** | Gen 0 (nursery) → Gen 1 (buffer) → Gen 2 (tenured). See [[Object Generations]] |
| **Collection phases** | Mark → Sweep → Compact. See [[Compaction Process]] |
| **Liveness** | Tracing from roots, not reference counting. See [[Determining if an Object is Live]] |
| **Triggers** | Budget exceeded, memory pressure, explicit `GC.Collect()` |
| **GC modes** | Workstation (latency) vs Server (throughput), Background GC for Gen 2 |
| **Finalization** | 2+ GC passes, cascade promotion. See [[Finalizable Objects]] |
| **Memory vs lifecycle** | GC handles memory only. Dispose handles lifecycle. See [[IDisposable]] |
