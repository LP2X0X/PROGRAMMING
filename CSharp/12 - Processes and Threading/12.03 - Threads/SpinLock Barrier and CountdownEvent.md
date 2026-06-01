---
tags:
 - csharp
 - threading
 - synchronization
---

These are specialized synchronization primitives for specific scenarios. You won't use them often, but they solve problems that the general-purpose tools (`lock`, `Semaphore`, etc.) handle less efficiently.


---

## SpinLock — Busy-Wait for Microsecond-Scale Locks

### The Problem with `lock` for Very Short Critical Sections

When a thread can't acquire a `lock`, it goes to **sleep** — the OS removes it from the CPU, saves its state, and schedules another thread. When the lock becomes available, the OS wakes it up. This context switch takes **~1-2 microseconds**.

If the critical section itself takes **less than a microsecond**, sleeping and waking costs more than just waiting. It's like driving to a mailbox across the street.

### The Solution — Spin

A `SpinLock` doesn't put the thread to sleep. Instead, the thread **busy-waits** in a tight loop ("spinning"), constantly checking if the lock is available. It burns CPU, but for extremely short waits, it's faster than a context switch.

```csharp
private SpinLock _spinLock = new(); // NOTE: SpinLock is a struct

void UpdateCounter()
{
    bool lockTaken = false;
    try
    {
        _spinLock.Enter(ref lockTaken);
        // Critical section — must be VERY short (nanoseconds to low microseconds)
        _counter++;
    }
    finally
    {
        if (lockTaken)
            _spinLock.Exit();
    }
}
```

### When to Use SpinLock

| Scenario | Use |
|---|---|
| Critical section < 1μs, high contention | `SpinLock` |
| Critical section > a few μs | `lock` — spinning wastes CPU |
| Not sure | `lock` — safer default |

```ad-warning
`SpinLock` is a **struct**. Don't pass it by value or it will be copied (and the copy is a separate, unlocked SpinLock). Always use `ref` or store it in a field. Also, never use it for long-held locks — a spinning thread burns 100% of a CPU core while waiting.
```


---

## Barrier — "Everyone Wait Here Until We're All Ready"

### The Problem

You have N threads doing work in **phases**. Each thread must finish phase 1 before any thread starts phase 2. You need a synchronization point where threads wait for each other.

### The Solution

A `Barrier` blocks all threads until a specified number of participants have arrived, then releases them all at once to start the next phase.

```csharp
// 3 threads must all reach the barrier before any proceed
var barrier = new Barrier(participantCount: 3, postPhaseAction: b =>
{
    Console.WriteLine($"--- Phase {b.CurrentPhaseNumber} complete ---");
});

for (int i = 0; i < 3; i++)
{
    int id = i;
    new Thread(() =>
    {
        // Phase 0
        Console.WriteLine($"Thread {id}: doing phase 0 work");
        Thread.Sleep(Random.Shared.Next(500, 1500));
        barrier.SignalAndWait(); // "I'm done — waiting for others"

        // Phase 1 — only starts after ALL threads finished phase 0
        Console.WriteLine($"Thread {id}: doing phase 1 work");
        Thread.Sleep(Random.Shared.Next(500, 1500));
        barrier.SignalAndWait();

        // Phase 2
        Console.WriteLine($"Thread {id}: doing phase 2 work");
    }).Start();
}
```

Output:

```
Thread 0: doing phase 0 work
Thread 2: doing phase 0 work
Thread 1: doing phase 0 work
--- Phase 0 complete ---
Thread 0: doing phase 1 work
Thread 1: doing phase 1 work
Thread 2: doing phase 1 work
--- Phase 1 complete ---
Thread 0: doing phase 2 work
Thread 2: doing phase 2 work
Thread 1: doing phase 2 work
```

Each thread can finish its phase at different times, but none starts the next phase until all have arrived at the barrier.

```
Phase 0                    Barrier              Phase 1
                              │
Thread A ─── work ──────────► │
Thread B ─── work ──► wait    │
Thread C ─── work ────────► wait
                              │ all arrived!
                              ▼
                    ALL released together
```

### Use Cases

- Parallel simulations where each step depends on the previous step's complete results
- Multi-threaded image processing where each thread handles a region, then they merge
- Testing scenarios where threads must synchronize at specific points


---

## CountdownEvent — "Wait Until N Things Have Happened"

### The Problem

You launch N parallel tasks and need to wait until **all N have completed** (or N events have occurred) before proceeding. `Task.WhenAll` works for `Task` objects, but what about raw threads or arbitrary signals?

### The Solution

`CountdownEvent` starts with a count and reaches zero as threads signal it. When it hits zero, any thread waiting on it is released.

```csharp
var countdown = new CountdownEvent(initialCount: 3);

for (int i = 0; i < 3; i++)
{
    int id = i;
    new Thread(() =>
    {
        Console.WriteLine($"Worker {id} doing work...");
        Thread.Sleep(Random.Shared.Next(500, 2000));
        Console.WriteLine($"Worker {id} done");
        countdown.Signal(); // decrement count by 1
    }).Start();
}

Console.WriteLine("Main: waiting for all workers...");
countdown.Wait(); // blocks until count reaches 0
Console.WriteLine("Main: all workers finished!");
```

### CountdownEvent vs Barrier

| Feature | CountdownEvent | Barrier |
|---|---|---|
| Direction | Counts **down** to zero, then releases | Threads **meet**, then all proceed together |
| Reusable | One-shot (must `Reset()` to reuse) | Automatically resets for the next phase |
| Phases | Single wait point | Multiple synchronized phases |
| Use case | "Wait for N things to finish" | "All threads must stay in sync across phases" |

### Adjusting the Count

```csharp
var countdown = new CountdownEvent(5);

// Oops, we're launching 2 more tasks
countdown.AddCount(2); // now 7

// Or one task was cancelled
countdown.Signal(2); // signal 2 at once (count drops by 2)
```

```ad-note
`CountdownEvent` is a simpler alternative to `Task.WhenAll` when you're working with raw threads or need to count arbitrary events (not just task completions). For `Task`-based code, prefer `Task.WhenAll`.
```


---

## Summary — When to Use Each

| Primitive | One-liner | Use when |
|---|---|---|
| `SpinLock` | Busy-wait lock for nanosecond-scale critical sections | Lock is held < 1μs, you want to avoid context switch overhead |
| `Barrier` | "Meet here, then proceed together" — repeatable | Threads work in phases and must synchronize between each phase |
| `CountdownEvent` | "Wait for N signals" — one-shot | You need to wait for N events/completions before proceeding |
