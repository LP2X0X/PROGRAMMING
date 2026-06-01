---
tags:
 - csharp
 - threading
 - clr-internals
---

## The Three Pieces

When you `lock` on an object, three things work together: the **object header**, the **Sync Block Table**, and the **thread ID**.

```
 OBJECT ON HEAP                    SYNC BLOCK TABLE (global CLR array)
┌──────────────────┐               ┌───────┬──────────┬───────┬────────────┐
│  Object Header   │               │ Index │ Owner ID │ Count │ Wait Queue │
│  SyncBlk Index: 7├──────────────►│   7   │ Thread A │   1   │  (empty)   │
├──────────────────┤               ├───────┼──────────┼───────┼────────────┤
│  Method Table Ptr│               │   8   │   ---    │   0   │   ---      │
├──────────────────┤               └───────┴──────────┴───────┴────────────┘
│  Your Fields     │
└──────────────────┘
```

- **Object header** — every reference type has one. It holds a small integer: the **sync block index**. This is NOT the lock state itself — it's a pointer (index) into a separate table.
- **Sync Block Table** — a global array maintained by the CLR. Each entry stores the actual lock state: owner thread ID, recursion count, and pointers to ready/wait queues.
- **Thread ID** — the managed thread ID of the thread that currently owns the lock. Used to distinguish "same thread re-entering" from "different thread trying to take the lock."


---

## Step by Step — First Lock on an Object

### Before Any Locking

The object's sync block index is unused (sentinel value). No table entry is allocated. **Zero overhead** for the vast majority of objects that are never locked.

### Thread A Calls `Monitor.Enter(obj)`

```
1. CLR reads obj's header → sync block index is empty
2. CLR allocates a NEW entry in the Sync Block Table → index 7
3. CLR writes 7 into the object's header
4. CLR writes Thread A's managed thread ID into entry 7's Owner field
5. CLR sets entry 7's Count to 1
```

The object now "knows" it's locked — but only indirectly. It points to a table entry, and that entry says "Thread A owns me."


---

## Contention — Thread B Tries the Same Lock

```
1. CLR reads obj's header → sync block index = 7
2. Looks up entry 7 in the table
3. Entry 7 says: Owner = Thread A's ID
4. Thread B's ID ≠ Thread A's ID → can't enter
5. Thread B is placed in the ready queue inside entry 7
6. Thread B sleeps (zero CPU)
```

```
  Sync Block Table Entry 7:
  ┌──────────────────────────────┐
  │  Owner: Thread A (ID=5)      │
  │  Count: 1                    │
  │  Ready Queue: [Thread B]     │
  └──────────────────────────────┘
```


---

## Release — Thread A Calls `Monitor.Exit(obj)`

```
1. CLR reads obj's header → index 7
2. Decrements Count: 1 → 0
3. Count hit 0 → CLR clears the Owner field
4. CLR checks entry 7's ready queue → Thread B is waiting
5. CLR wakes Thread B
6. CLR writes Thread B's ID into entry 7's Owner field
7. CLR sets Count back to 1
```

The entry is **reused** — the object header still says index 7, but now entry 7 says Thread B owns the lock.

```
  Sync Block Table Entry 7 (after handoff):
  ┌──────────────────────────────┐
  │  Owner: Thread B (ID=9)      │
  │  Count: 1                    │
  │  Ready Queue: (empty)        │
  └──────────────────────────────┘
```


---

## Reentrancy — Same Thread Re-enters

If Thread A calls `lock(obj)` while already inside a `lock(obj)`:

```
First  Monitor.Enter → Owner = Thread A, Count = 1
Second Monitor.Enter → Owner matches Thread A's ID → Count = 2 (enter immediately)
First  Monitor.Exit  → Count = 1 (still locked!)
Second Monitor.Exit  → Count = 0 → release, wake waiters
```

The CLR stores the owner thread ID specifically to answer:
- **"Is this a different thread?"** → block it (contention)
- **"Is this the same thread re-entering?"** → increment count, let it in (reentrancy)

Without the thread ID, the CLR couldn't tell these apart.


---

## Why This Design — Pay-for-Play

Most objects are **never locked**. If lock state lived directly in every object's header, you'd waste memory on millions of objects that never need it.

Instead:
- **Unlocked objects** — header is just one small unused integer. Minimal cost.
- **Locked objects** — one integer in the header + one table entry. Allocated on demand.

The Sync Block Table is the CLR's way of saying "I'll only allocate lock infrastructure for objects that actually need it."

See [[Lock and Monitor]] for usage patterns, deadlocks, and when to use `lock` vs other synchronization primitives.
