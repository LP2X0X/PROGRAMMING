---
tags:
 - csharp
 - threading
 - synchronization
---

## The Problem

`lock` and `Mutex` are binary — one thread in, everyone else waits. But sometimes you want to allow **a limited number** of threads to access a resource concurrently. For example:

- A database connection pool with 10 connections — up to 10 threads can use it, the 11th must wait
- An API with a rate limit — only 5 requests at a time
- A file system where you want to limit concurrent I/O to avoid thrashing

A **Semaphore** is a counter-based lock. You set a maximum count, and that many threads can enter simultaneously. When the count reaches 0, the next thread blocks until someone releases.


---

## How It Works — The Bouncer Analogy

Think of a semaphore as a bouncer at a club with a capacity limit:

```
Capacity: 3

Thread A → enters (count: 2 remaining)
Thread B → enters (count: 1 remaining)
Thread C → enters (count: 0 remaining)
Thread D → BLOCKED — waits at the door
Thread A → leaves (count: 1 remaining)
Thread D → enters (count: 0 remaining)
```

Each `WaitOne()` / `Wait()` decrements the count. Each `Release()` increments it. When the count is 0, threads block.


---

## SemaphoreSlim — The One You Should Use

`SemaphoreSlim` is the lightweight, in-process version. Use this by default:

```csharp
// Allow up to 3 concurrent threads
private readonly SemaphoreSlim _semaphore = new(initialCount: 3, maxCount: 3);

async Task AccessLimitedResource(int id)
{
    Console.WriteLine($"Thread {id} waiting...");
    await _semaphore.WaitAsync(); // async-friendly!
    try
    {
        Console.WriteLine($"Thread {id} entered (slots left: {_semaphore.CurrentCount})");
        await Task.Delay(2000); // simulate work
    }
    finally
    {
        _semaphore.Release();
        Console.WriteLine($"Thread {id} released");
    }
}

// Launch 6 tasks — only 3 run at a time
var tasks = Enumerable.Range(1, 6).Select(i => AccessLimitedResource(i));
await Task.WhenAll(tasks);
```

Output (approximate):

```
Thread 1 entered (slots left: 2)
Thread 2 entered (slots left: 1)
Thread 3 entered (slots left: 0)
Thread 4 waiting...
Thread 5 waiting...
Thread 6 waiting...
Thread 1 released
Thread 4 entered (slots left: 0)
...
```

### Constructor Parameters

```csharp
new SemaphoreSlim(initialCount: 3, maxCount: 3);
```

| Parameter | Meaning |
|---|---|
| `initialCount` | How many slots are available right now |
| `maxCount` | The absolute maximum — `Release()` will throw if you exceed this |

Setting `initialCount` lower than `maxCount` means some slots start occupied:

```csharp
// Max 5, but start with only 2 available (3 slots "pre-reserved")
new SemaphoreSlim(initialCount: 2, maxCount: 5);
```


---

## SemaphoreSlim as an Async Lock

With `initialCount: 1`, a `SemaphoreSlim` acts like an **async-compatible lock** — something `lock` can't do because you can't `await` inside a `lock` block:

```csharp
private readonly SemaphoreSlim _asyncLock = new(1, 1);

async Task SafeUpdateAsync()
{
    await _asyncLock.WaitAsync();
    try
    {
        // Only one thread at a time — like lock, but async-safe
        await SomeAsyncDatabaseCall();
    }
    finally
    {
        _asyncLock.Release();
    }
}
```

```ad-note
You **cannot** use `await` inside a `lock` block — the compiler will give you an error. `SemaphoreSlim(1, 1)` with `WaitAsync()` is the standard solution for async mutual exclusion.
```


---

## Semaphore (Full) — Cross-Process

The full `Semaphore` class (not Slim) can work across processes using a name, just like `Mutex`:

```csharp
// System-wide: only 3 processes can hold this at once
using var sem = new Semaphore(
    initialCount: 3,
    maximumCount: 3,
    name: "Global\\MyApp_DbPool");

sem.WaitOne(); // blocks if 3 others already hold it
try
{
    DoWork();
}
finally
{
    sem.Release();
}
```


---

## Semaphore vs SemaphoreSlim

| Feature | `Semaphore` | `SemaphoreSlim` |
|---|---|---|
| Cross-process (named) | Yes | No |
| `WaitAsync()` (async support) | No | **Yes** |
| Performance | Slow (OS kernel call) | Fast (user-mode) |
| `CurrentCount` property | No | Yes |
| Use when | Cross-process throttling | Everything else (default choice) |


---

## Common Patterns

### Rate Limiting API Calls

```csharp
private readonly SemaphoreSlim _apiThrottle = new(5, 5);

async Task<string> CallApiThrottled(string url)
{
    await _apiThrottle.WaitAsync();
    try
    {
        return await _httpClient.GetStringAsync(url);
    }
    finally
    {
        _apiThrottle.Release();
    }
}
```

### Connection Pool

```csharp
private readonly SemaphoreSlim _pool = new(10, 10);

async Task UseConnectionAsync()
{
    await _pool.WaitAsync();
    try
    {
        var conn = GetConnectionFromPool();
        await conn.ExecuteQueryAsync();
        ReturnToPool(conn);
    }
    finally
    {
        _pool.Release();
    }
}
```

```ad-warning
Always call `Release()` in a `finally` block. If a thread crashes without releasing, the semaphore count is permanently reduced — eventually all slots are "leaked" and every thread blocks forever.
```
