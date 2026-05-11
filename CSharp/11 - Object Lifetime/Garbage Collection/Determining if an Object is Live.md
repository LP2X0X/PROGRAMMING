---
tags:
 - csharp
 - object-lifetime
 - gc
---

## How the GC Determines Liveness

The GC uses **tracing**, not reference counting. It starts from a set of **roots** and walks every reference chain. Anything it can reach is live — everything else is garbage.

### GC Roots

- **Stack roots** — local variables and method parameters on the current call stack
- **Static fields** — static objects that live for the lifetime of the application domain
- **GC handles** — handles that point to managed objects referenced from code or the runtime (e.g., pinned handles for native interop)
- **CPU registers** — references held in registers at the time of collection

### The Object Graph (Tracing)

During collection, the GC builds an **object graph** starting from all roots:

1. Mark every root as reachable
2. Follow each reference from a reachable object → mark the target as reachable too
3. Repeat recursively until no new objects are found
4. Anything **not** marked is unreachable → garbage

```
Roots (stack, statics, registers)
  │
  ├──→ ObjA ──→ ObjB ──→ ObjC     ← all reachable (live)
  │
  └──→ ObjD                        ← reachable (live)

  ObjE ←──→ ObjF                   ← circular reference, but no root
                                      can reach either → both collected
```

The GC never visits the same object twice during tracing — this avoids infinite loops from circular references. This is why .NET doesn't have the circular reference problem that COM reference counting had.

![[Pasted image 20240605093217.png|center]]
![[Pasted image 20240605093228.png|center]]

## The Three Heaps

| Heap | What goes there | Compacted after collection? |
|---|---|---|
| **SOH** (Small Object Heap) | Objects < 85,000 bytes | Yes — surviving objects are moved together to eliminate gaps |
| **LOH** (Large Object Heap) | Objects >= 85,000 bytes | No by default — moving large blocks is expensive |
| **POH** (Pinned Object Heap, .NET 5+) | Objects pinned for native interop | No — native code holds raw pointers, so these can't move |

### Why compaction matters

After the GC sweeps dead objects, the SOH has gaps. Compaction slides surviving objects together so new allocations can just go at the end (fast bump-pointer allocation):

```
SOH before compaction:
[ObjA][  dead  ][ObjB][  dead  ][ObjC]   ← fragmented

SOH after compaction:
[ObjA][ObjB][ObjC][      free space     ] ← clean
```

The LOH skips compaction because copying large blocks is slow. This means it can fragment over time. You can force LOH compaction when needed:

```csharp
GCSettings.LargeObjectHeapCompactionMode = GCLargeObjectHeapCompactionMode.CompactOnce;
GC.Collect();
```

The POH (added in .NET 5) solves a different problem — pinned objects on the SOH blocked compaction around them, causing SOH fragmentation. Moving them to their own heap keeps the SOH clean.


