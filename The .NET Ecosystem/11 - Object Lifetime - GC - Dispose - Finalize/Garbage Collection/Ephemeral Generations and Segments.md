---
tags:
 - csharp
 - object-lifetime
 - keyword
---

**Ephemeral** means short-lived — and that's exactly what Gen 0 and Gen 1 are. The GC groups them together under this term because they share a key property: objects in these generations are expected to die quickly, so the GC treats them differently from Gen 2.

## Ephemeral Generations

Generations 0 and 1 are called **ephemeral generations**. They are:
- **Small** — intentionally sized to fit in CPU cache for fast scanning.
- **Collected frequently** — Gen 0 collections are the most common GC event.
- **Cheap to collect** — because they're small and most objects in them are already dead.

Gen 2 is **not** ephemeral. It's the long-lived / tenured generation.

## Ephemeral Segment

The ephemeral generations live in a memory segment called the **ephemeral segment**. This is a contiguous block of memory that the GC allocates from the OS.

- Gen 0 and Gen 1 always live in the **same segment** — the current ephemeral segment.
- New object allocations happen at the end of the ephemeral segment (in Gen 0). This is extremely fast — it's just bumping a pointer forward.
- When a GC occurs and objects survive past Gen 1 into Gen 2, the current ephemeral segment may become a **Gen 2 segment**, and the GC acquires a **new ephemeral segment** for future Gen 0/Gen 1 allocations.

```
Ephemeral Segment (single contiguous block)
┌─────────────────────────────────────────────────────┐
│      Gen 1       │              Gen 0               │ ← allocation pointer
└─────────────────────────────────────────────────────┘
                                                      ↑
                                          new objects go here

After collection (survivors promoted):

Old segment becomes Gen 2        New Ephemeral Segment
┌──────────────────────┐        ┌──────────────────────┐
│       Gen 2          │        │  Gen 1  │   Gen 0    │
└──────────────────────┘        └──────────────────────┘
```

The reason the GC keeps ephemeral generations in their own segment is performance — it can collect Gen 0 and Gen 1 without touching Gen 2's memory at all, which keeps cache locality tight and pause times low.

![[Pasted image 20240605104747.png|center]]
