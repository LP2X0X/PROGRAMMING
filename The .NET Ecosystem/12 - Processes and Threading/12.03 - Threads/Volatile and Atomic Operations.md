---
tags:
 - csharp
 - threads
 - concurrency
---

## The Problem — Why Shared Data Breaks

When two threads read and write the same variable, two invisible things can go wrong — even before you think about race conditions.


---

## Problem 1: Caching — The CPU Lies to You

Modern CPUs don't read from main memory every time. Each core has its own **cache** — a private, fast copy of recently accessed memory. When a thread writes to a variable, it may only update **its own core's cache**, not main memory. Another thread on a different core may keep reading the **stale value** from its own cache indefinitely.

```csharp
// Thread 1 (on Core A)
_running = true;
while (_running)
{
    DoWork();
}

// Thread 2 (on Core B)
_running = false;  // writes to Core B's cache
// Thread 1 may NEVER see this — it's reading from Core A's cache
```

This is not a timing issue. Thread 1 can loop **forever** because the CPU optimized away the memory read. It "knows" `_running` didn't change *from its perspective*.


---

## Problem 2: Reordering — The Compiler and CPU Rearrange Your Code

Both the **C# compiler** (JIT) and the **CPU** can reorder instructions for performance, as long as the result looks the same *from a single thread's perspective*. But with multiple threads watching the same memory, this breaks assumptions:

```csharp
// You wrote this:
_data = 42;
_ready = true;

// The CPU/compiler might execute this:
_ready = true;     // moved up!
_data = 42;

// Another thread sees _ready == true but _data is still 0
```

From thread 1's perspective, the result is the same — both assignments happen. But thread 2 might check `_ready`, see `true`, and read `_data` before it's been written.


---

## Solution 1: `volatile` — Prevent Caching and Restrict Reordering

The `volatile` keyword tells the compiler and CPU:
- **Every read** of this field must go to main memory (no cached copies)
- **Every write** must flush to main memory immediately
- Reads and writes cannot be reordered past this point

```csharp
class Worker
{
    private volatile bool _running = true;

    public void DoWork()
    {
        // Guaranteed to re-read _running from memory each iteration
        while (_running)
        {
            // work...
        }
    }

    public void Stop()
    {
        // Guaranteed to be visible to other threads immediately
        _running = false;
    }
}
```

### What `volatile` Can Be Used On

`volatile` only works on fields that can be read/written atomically in a single CPU operation:

| Allowed | Not allowed |
|---|---|
| `bool`, `byte`, `short`, `int`, `char`, `float` | `long`, `double`, `decimal` |
| Reference types (the pointer, not the object) | Structs |
| Enums (if backed by int or smaller) | |

```ad-warning
`volatile` does NOT make operations atomic. `_counter++` on a volatile field is still broken — it's a read-modify-write (three steps). `volatile` only guarantees individual reads and writes are not cached or reordered.
```


---

## Solution 2: `Interlocked` — True Atomic Operations

Some operations look like one step in code but are actually **multiple CPU instructions**. The classic example:

```csharp
_counter++;

// What the CPU actually does:
// 1. READ _counter from memory  → temp = 5
// 2. ADD 1                      → temp = 6
// 3. WRITE back to memory       → _counter = 6

// If two threads do this at the same time:
// Thread A: READ → 5    Thread B: READ → 5
// Thread A: ADD  → 6    Thread B: ADD  → 6
// Thread A: WRITE → 6   Thread B: WRITE → 6
// Expected: 7. Actual: 6. One increment was lost.
```

This is a **race condition**. The `Interlocked` class solves it by performing read-modify-write as a **single, uninterruptible CPU instruction**:

```csharp
// Atomic increment — no other thread can interfere mid-operation
Interlocked.Increment(ref _counter);

// Atomic decrement
Interlocked.Decrement(ref _counter);

// Atomic add
Interlocked.Add(ref _counter, 10);

// Atomic swap — sets new value, returns old value
int old = Interlocked.Exchange(ref _counter, 0);

// Atomic compare-and-swap — only writes if current value matches expected
Interlocked.CompareExchange(ref _counter, newValue: 10, comparand: 5);
// If _counter == 5, set it to 10. Otherwise, do nothing.
```

`Interlocked` also handles memory barriers automatically — you don't need `volatile` on fields you access through `Interlocked`.

### When to Use `Interlocked`

Use it when your entire critical section is **a single read-modify-write on a single variable**. If you only need to protect one counter, one flag, or one reference swap — `Interlocked` is the right tool.

Why not just use `lock`? Because `lock` involves a sync block lookup, thread ID check, and if contended: kernel transition, thread sleep/wake, and context switch. For a single increment, that's massive overhead. `Interlocked` compiles to **one CPU instruction** with a hardware `lock` prefix — no lock object, no sync block, no kernel call.

### Practical Use Cases

**Counters:**

```csharp
Interlocked.Increment(ref _activeRequests);   // request started
Interlocked.Decrement(ref _activeRequests);   // request ended
```

**Unique ID generation:**

```csharp
// Every thread gets a unique ID — Increment returns the NEW value
long newId = Interlocked.Increment(ref _nextId);
```

**Set a flag exactly once (dispose guard):**

```csharp
// CompareExchange: "if _disposed is currently 0, set it to 1"
// Returns the ORIGINAL value. First thread gets 0 (success), all others get 1 (already done).
if (Interlocked.CompareExchange(ref _disposed, 1, 0) == 0)
{
    CleanUpResources(); // only one thread ever reaches here
}
```

**Atomic reference swap (hot-swap a config):**

```csharp
// Atomically swap the reference — readers see old or new, never torn
Interlocked.Exchange(ref _currentConfig, newConfig);
```

**CompareExchange loop (building block of lock-free code):**

```csharp
// Atomic add for double — no built-in Interlocked.Add for double
double initial, computed;
do
{
    initial = Volatile.Read(ref _total);
    computed = initial + value;
} while (Interlocked.CompareExchange(ref _total, computed, initial) != initial);
// If another thread changed _total between read and swap, loop retries
```

### When NOT to Use `Interlocked`

When you need **multiple variables** to change together as one unit:

```csharp
// WRONG — Interlocked can't make two operations atomic together
Interlocked.Add(ref from.Balance, -amount);
// crash here → money vanishes
Interlocked.Add(ref to.Balance, amount);

// RIGHT — multiple steps need lock
lock (_lock)
{
    from.Balance -= amount;
    to.Balance += amount;
}
```

`Interlocked` protects **one operation** on **one variable**. The moment you need two things to happen together, you need `lock`. See [[Lock and Monitor]].


---

## `volatile` vs `Interlocked` vs `lock`

| Mechanism | What it does | Use when |
|---|---|---|
| `volatile` | Prevents caching/reordering of single reads and writes | Simple flags (`bool _running`, `bool _ready`) |
| `Interlocked` | Makes read-modify-write operations atomic | Counters, accumulators, atomic swaps |
| `lock` | Ensures only one thread executes a block at a time | Multiple related operations that must be consistent together |

```csharp
// volatile — good for a simple flag
private volatile bool _done = false;

// Interlocked — good for a counter
Interlocked.Increment(ref _count);

// lock — good for multiple operations that must be consistent
lock (_lockObj)
{
    var item = _queue.Dequeue();
    _processing.Add(item);
    _count--;
}
```

Each level solves a bigger problem but costs more. Use the lightest tool that covers your case.


---

## The Hierarchy of Thread Safety

```
             More protection, more overhead
                        ▲
                        │
              ┌─────────┴─────────┐
              │      lock         │  Multiple operations as one unit
              │ (Monitor.Enter)   │
              └─────────┬─────────┘
                        │
              ┌─────────┴─────────┐
              │   Interlocked     │  Single operation, atomic
              │ (hardware-level)  │
              └─────────┬─────────┘
                        │
              ┌─────────┴─────────┐
              │    volatile       │  Single read or write, no caching
              └─────────┬─────────┘
                        │
              ┌─────────┴─────────┐
              │   No protection   │  Only safe if single-threaded
              └───────────────────┘
```

```ad-note
Most application code should prefer `lock` for clarity and safety. `volatile` and `Interlocked` are lower-level tools used in performance-critical code or within framework/library internals where the overhead of `lock` matters. When in doubt, use `lock`. See [[Lock and Monitor]] for a deep dive on `lock` and `Monitor`.
```
