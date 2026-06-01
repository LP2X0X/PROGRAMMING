---
tags:
 - csharp
 - threading
 - synchronization
---

## The Problem

Locks (`lock`, `Mutex`, `Semaphore`) protect shared data — they keep threads **out** of critical sections. But sometimes you need the opposite: a way for one thread to **tell another thread** that something happened. Not "stay out" — but "go ahead."

Examples:
- Thread A prepares data, then signals thread B that the data is ready
- A background worker waits for a "start" signal before beginning
- Multiple threads wait until initialization is complete

This is **signaling** — one thread raises a flag, other threads wait for that flag.


---

## The Concept — A Gate

Think of a signaling primitive as a **gate**:

- **Unsignaled (closed)** — any thread that calls `WaitOne()` blocks and sleeps
- **Signaled (open)** — threads calling `WaitOne()` pass through immediately

The difference between the two types is what happens **after** a thread passes through:


---

## ManualResetEvent — A Gate That Stays Open

When you `Set()` it, the gate opens and **stays open**. Every thread that calls `WaitOne()` passes through immediately — until you explicitly `Reset()` it.

```csharp
var gate = new ManualResetEvent(initialState: false); // starts closed

// Worker threads — all waiting for the signal
for (int i = 0; i < 3; i++)
{
    int id = i;
    new Thread(() =>
    {
        Console.WriteLine($"Worker {id} waiting...");
        gate.WaitOne(); // blocks until gate is opened
        Console.WriteLine($"Worker {id} proceeding!");
    }).Start();
}

Thread.Sleep(2000); // simulate setup work

Console.WriteLine("Main: opening the gate!");
gate.Set(); // ALL waiting threads are released at once

// Output:
// Worker 0 waiting...
// Worker 1 waiting...
// Worker 2 waiting...
// Main: opening the gate!
// Worker 0 proceeding!
// Worker 1 proceeding!
// Worker 2 proceeding!
```

The gate stays open — even threads that call `WaitOne()` **after** `Set()` pass through immediately. To close it again, call `Reset()`.

```
ManualResetEvent:
  Set()    →  gate opens  →  ALL waiters released  →  stays open
  Reset()  →  gate closes →  future waiters block
```


---

## AutoResetEvent — A Turnstile

When you `Set()` it, the gate opens for **exactly one** thread, then automatically closes again. It's a turnstile, not a gate.

```csharp
var turnstile = new AutoResetEvent(initialState: false);

// Three workers waiting
for (int i = 0; i < 3; i++)
{
    int id = i;
    new Thread(() =>
    {
        Console.WriteLine($"Worker {id} waiting...");
        turnstile.WaitOne();
        Console.WriteLine($"Worker {id} proceeding!");
    }).Start();
}

Thread.Sleep(1000);

// Release them one at a time
turnstile.Set(); // releases ONE worker
Thread.Sleep(500);
turnstile.Set(); // releases another ONE
Thread.Sleep(500);
turnstile.Set(); // releases the last one

// Output (order may vary):
// Worker 0 waiting...
// Worker 1 waiting...
// Worker 2 waiting...
// Worker 0 proceeding!
// Worker 2 proceeding!
// Worker 1 proceeding!
```

```
AutoResetEvent:
  Set()  →  gate opens  →  ONE waiter released  →  automatically closes
  Set()  →  gate opens  →  ONE waiter released  →  automatically closes
```


---

## ManualResetEvent vs AutoResetEvent

| Feature | ManualResetEvent | AutoResetEvent |
|---|---|---|
| `Set()` releases | **All** waiting threads | **One** waiting thread |
| After release | Stays open (must call `Reset()`) | Automatically resets to closed |
| Analogy | Gate — open/close manually | Turnstile — lets one through, closes |
| Use case | "Initialization is done, everyone proceed" | "Here's one unit of work, one thread take it" |


---

## The Slim Variants — ManualResetEventSlim

`ManualResetEventSlim` is the lightweight, in-process-only version (similar to `SemaphoreSlim` vs `Semaphore`):

```csharp
var gate = new ManualResetEventSlim(initialState: false);

// Waiting
gate.Wait();             // blocks (not WaitOne)
gate.Wait(timeout);      // blocks with timeout
// gate has no WaitAsync — use TaskCompletionSource for async signaling

// Signaling
gate.Set();
gate.Reset();
```

| Feature | `ManualResetEvent` | `ManualResetEventSlim` |
|---|---|---|
| Cross-process (named) | Yes | No |
| Performance | Slower (OS kernel) | Faster (user-mode with spin-wait) |
| Method names | `WaitOne()`, `Set()`, `Reset()` | `Wait()`, `Set()`, `Reset()` |
| Use when | Cross-process signaling | Everything else (default choice) |

```ad-note
There is no `AutoResetEventSlim`. If you need an auto-reset pattern with slim performance, use `SemaphoreSlim(0, 1)` — calling `Release()` lets one waiter through, then it's back to 0.
```


---

## Practical Pattern — Wait for Initialization

```csharp
class Service
{
    private readonly ManualResetEventSlim _initialized = new(false);
    private Config _config;

    public void Initialize()
    {
        _config = LoadConfigFromDatabase(); // slow
        _initialized.Set(); // signal: "config is ready"
    }

    public Config GetConfig()
    {
        _initialized.Wait(); // blocks until Initialize() has been called
        return _config;
    }
}
```

Any number of threads can call `GetConfig()`. If initialization isn't done yet, they all block. Once `Set()` is called, they all proceed — and future callers pass through instantly because the gate stays open.
