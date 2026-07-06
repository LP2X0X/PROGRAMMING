---
tags:
 - csharp
 - object-lifetime
 - gc
---

Compaction is the third and final phase of garbage collection (after Mark and Sweep). It physically moves surviving objects on the **Small Object Heap (SOH)** to eliminate gaps left by dead objects, producing a contiguous block of free memory. This keeps allocation fast and prevents heap fragmentation from degrading performance over time.

---

## Why Compaction Exists

When the GC sweeps dead objects, the SOH ends up with holes scattered throughout:

```
Before compaction:
[ObjA][  dead  ][ObjB][ dead ][ dead ][ObjC][ObjD][  dead  ]

After compaction:
[ObjA][ObjB][ObjC][ObjD][          free space              ]
```

Without compaction, the allocator would need to search through these gaps to find a block large enough for each new object — a **free list** approach, which is slower and leads to fragmentation over time. With compaction, the allocator uses a **bump pointer** — it simply increments a pointer to allocate the next object. This is nearly as fast as stack allocation.

---

## How Compaction Works — Step by Step

### Step 1: Identify Gaps

After the sweep phase has identified dead objects, the GC knows exactly where the gaps are in the heap. It builds a table of gap locations and sizes.

### Step 2: Calculate New Addresses

The GC computes where each surviving object will end up after compaction. It walks through the heap linearly, assigning new positions by packing objects tightly together, skipping over gaps.

### Step 3: Relocate Objects (Object Copying)

The GC physically copies each surviving object to its new position. This is done efficiently using **bulk memory copy** operations (`memcpy`-style) rather than object-by-object field copying. Modern CPUs are heavily optimized for sequential memory operations, and the GC takes advantage of this.

```
Source:      [ObjA][____][ObjB][____][____][ObjC]
                          ↓                 ↓
Destination: [ObjA][ObjB][ObjC][                ]
```

### Step 4: Update References (Pointer Fixup)

After objects have moved, every reference in the entire managed heap and all GC roots that pointed to a moved object must be updated to reflect the new address. This is the most complex part of compaction:

- **Stack roots** — local variables and parameters on every thread's call stack
- **Static fields** — all static references across all loaded types
- **Object fields** — every reference field inside every surviving object
- **GC handles** — runtime-managed handles (pinned handles, weak references, etc.)
- **Internal runtime structures** — method tables, thread-local storage, etc.

The GC uses the address mapping table from Step 2 to update each reference. After this step, all references point to the correct new locations and the application can resume.

---

## What Gets Compacted and What Doesn't

| Heap | Compacted? | Why |
|---|---|---|
| **SOH** (Gen 0, Gen 1, Gen 2) | Yes | Small objects — copying is fast, fragmentation is costly |
| **LOH** (Large Object Heap) | No by default | Copying multi-MB objects is expensive, causes long pauses |
| **POH** (Pinned Object Heap) | No | Native code holds raw pointers — objects cannot move |

### LOH Compaction (Opt-In)

The LOH uses a **free-list** allocator instead of bump-pointer. Over time, this can fragment. Since .NET 4.5.1, you can request LOH compaction explicitly:

```csharp
GCSettings.LargeObjectHeapCompactionMode = GCLargeObjectHeapCompactionMode.CompactOnce;
GC.Collect();
```

This compacts the LOH on the **next** full GC, then resets to the default (no compaction). It's a one-shot — you must set it again each time you want LOH compaction. Use this sparingly: LOH compaction is expensive because it moves very large objects.

### Pinned Objects Block Compaction

When an object is **pinned** (via `fixed` statement, `GCHandle.Alloc` with `GCHandleType.Pinned`, or `GC.AllocateArray<T>(pinned: true)`), the GC cannot move it. During SOH compaction, the GC must work around pinned objects — it compacts the live objects on either side but leaves the pinned object in place. This creates fragmentation around pinned objects.

```
Before compaction (P = pinned):
[ObjA][dead][P_pinned][dead][ObjB][dead][ObjC]

After compaction:
[ObjA][gap ][P_pinned][ObjB][ObjC][free       ]
       ↑
       Can't fill — P can't move
```

This is why .NET 5 introduced the **Pinned Object Heap (POH)** — see [[Determining if an Object is Live]] — to move pinned objects off the SOH entirely, keeping SOH compaction clean.

---

## Performance Cost of Compaction

Compaction is not free. It involves:

1. **Copying memory** — every surviving object is physically moved
2. **Updating references** — every pointer to a moved object must be fixed up
3. **Thread suspension** — compaction requires a "stop the world" pause because objects are being moved

However, the runtime uses several strategies to keep this cost manageable:

### Generational Collection Minimizes Work

Compaction typically affects only the **generation being collected**:

- **Gen 0 compaction** — the most frequent, but Gen 0 is tiny (typically 256 KB to a few MB, designed to fit in CPU cache). Compacting it is very fast.
- **Gen 1 compaction** — also small and cheap.
- **Gen 2 compaction** — rare and expensive. The GC avoids this by using background GC for the mark phase and only compacting when fragmentation is significant.

Since most objects die in Gen 0, most compaction work is on a small region of memory. See [[Object Generations]] for how the generational model reduces collection cost.

### Ephemeral Segment Optimization

Gen 0 and Gen 1 share a dedicated **ephemeral segment** — a contiguous block of memory sized to fit in CPU cache. Compaction within this segment is fast because:

- The data is likely already in cache
- The memory region is small
- Sequential access patterns are optimal for modern CPUs

See [[Ephemeral Generations and Segments]] for details.

### Parallel GC

In **Server GC** mode, the runtime creates one heap per logical CPU core. During compaction, each heap is compacted by its own dedicated GC thread in parallel — the work is distributed across cores.

### Background GC

**Background GC** (enabled by default) runs the Gen 2 mark phase concurrently with your application on a background thread. Only the final compaction/sweep step requires a brief stop-the-world pause, significantly reducing the total pause time for full collections.

### Efficient Bulk Copying

The runtime uses optimized bulk memory copy operations (similar to `memcpy` / `Buffer.BlockCopy`) rather than copying object fields individually. Modern CPUs with large cache lines and hardware prefetching handle sequential memory copies very efficiently.

### Adaptive Decisions

The GC doesn't always compact. It uses heuristics to decide whether compaction is worthwhile for a given collection:

- **Fragmentation level** — if the heap isn't significantly fragmented, the GC may skip compaction and just sweep (free-list mode for that region).
- **Survival rate** — if most objects survived, there are few gaps, so compaction would move many objects for little benefit.
- **Pause time budget** — the GC balances compaction benefit against the pause time it would cause.

This means not every GC cycle involves compaction — only when the runtime determines the benefit outweighs the cost.

---

## Benefits of Compaction

### 1. Bump-Pointer Allocation

After compaction, all free memory is at the end of the segment. Allocating a new object is just:

```
allocation_pointer += object_size;
```

No searching through free lists, no fragmentation checks. This makes managed allocation **faster than `malloc`** in most cases.

### 2. Reduced Fragmentation

Without compaction, long-running applications accumulate gaps that may be too small for new objects, wasting memory. Compaction eliminates this problem on the SOH.

### 3. Improved Cache Locality

Compaction places objects that were allocated around the same time (and often used together) into adjacent memory. This improves **spatial locality**, which means fewer CPU cache misses when your code traverses object graphs.

### 4. Predictable Allocation Performance

With bump-pointer allocation, every allocation takes roughly the same time. Without compaction, allocation time varies depending on free list state and fragmentation — harder to predict, harder to optimize.

---

## Compaction and Finalizable Objects

Finalizable objects interact with compaction in an important way. When the GC finds an unreachable finalizable object:

1. It moves the object to the **f-reachable queue** — making it reachable again.
2. The object **survives** the current collection and gets **promoted** to the next generation.
3. During compaction, this object is treated as a **survivor** — it gets moved along with other live objects.
4. Everything the finalizable object references also survives and is compacted into the higher generation.

This means finalizable objects force the GC to compact more data than necessary — one more reason to avoid unnecessary finalizers. See [[Finalizable Objects]] for the full cascade effect.

---

## Monitoring Compaction

You can observe GC behavior and compaction through:

- **dotnet-counters** — `gc-heap-size`, `gen-0-gc-count`, `gen-1-gc-count`, `gen-2-gc-count`
- **PerfView** — detailed GC event traces showing mark, sweep, compact durations
- **Visual Studio Diagnostic Tools** — real-time heap snapshots and GC events
- **ETW (Event Tracing for Windows)** — low-level GC events including compaction decisions

```csharp
// Check collection counts to understand GC frequency
Console.WriteLine($"Gen 0: {GC.CollectionCount(0)}");
Console.WriteLine($"Gen 1: {GC.CollectionCount(1)}");
Console.WriteLine($"Gen 2: {GC.CollectionCount(2)}");

// A healthy ratio looks like: Gen0 >> Gen1 >> Gen2
// e.g., 500 : 40 : 3
```

See [[System.GC]] for the programmatic API.

---

## Summary

| Aspect           | Detail                                                                                       |
| ---------------- | -------------------------------------------------------------------------------------------- |
| **What it does** | Slides surviving SOH objects together, eliminating gaps                                      |
| **Why**          | Enables fast bump-pointer allocation, prevents fragmentation                                 |
| **SOH**          | Always compacted (Gen 0, 1, 2)                                                               |
| **LOH**          | Not compacted by default (opt-in with `GCLargeObjectHeapCompactionMode.CompactOnce`)         |
| **POH**          | Never compacted (pinned objects can't move)                                                  | 
| **Cost**         | Memory copying + reference fixup + thread pause                                              |
| **Mitigations**  | Generational collection, ephemeral segments, parallel GC, background GC, adaptive heuristics |
| **Benefit**      | Near-instant allocation, cache locality, no fragmentation                                    |
