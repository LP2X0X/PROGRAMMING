---
tags:
 - csharp
 - threading
 - tpl
---

## What Is a Task?

A `Task` represents a **unit of work** that may or may not have completed yet. It's the central abstraction of the Task Parallel Library (TPL) — a higher-level way to work with concurrency than raw `Thread` objects.

Think of a `Task` as a **promise**: "this work will produce a result (or finish) at some point in the future. Here's a token you can use to check on it, wait for it, or chain more work after it."

```csharp
using System.Threading.Tasks;
```

### Task vs Thread

| | `Thread` | `Task` |
|---|---|---|
| Abstraction level | Low — wraps an OS thread directly | High — represents work, not a thread |
| Return value | None (`ThreadStart` is `void`) | `Task<T>` returns a value |
| Exception handling | Unhandled = process crash | Captured and re-thrown on `await`/`.Result` |
| Scheduling | You manage the thread | Runtime schedules on the thread pool |
| Composition | `Join()` only | `WhenAll`, `WhenAny`, `ContinueWith`, `await` |
| Cancellation | `Abort()` (removed in .NET 5+) | Cooperative via `CancellationToken` |
| Cost | ~1 MB stack + OS kernel object | Lightweight object on the managed heap |

A `Task` doesn't necessarily create a new thread. It **schedules work** on the thread pool. Multiple tasks share a small pool of threads.


---

## Creating and Running Tasks

### `Task.Run` — The Standard Way

Queues work on the thread pool and returns a `Task`:

```csharp
// Fire-and-forget (but awaitable)
Task work = Task.Run(() =>
{
    Console.WriteLine($"Running on thread {Thread.CurrentThread.ManagedThreadId}");
    Thread.Sleep(1000);
    Console.WriteLine("Done");
});

await work; // wait for it to finish
```

### `Task.Run` with a Return Value — `Task<T>`

```csharp
Task<int> computation = Task.Run(() =>
{
    int sum = 0;
    for (int i = 0; i < 1_000_000; i++)
        sum += i;
    return sum;
});

int result = await computation; // get the result when done
Console.WriteLine($"Sum: {result}");
```

### Constructor + `Start()` — Manual Control (Rare)

```csharp
var task = new Task(() => Console.WriteLine("Manual task"));
// task is in Created state — NOT running yet
task.Start(); // now it's queued on the thread pool
await task;
```

```ad-warning
Prefer `Task.Run` over `new Task()` + `Start()`. The constructor pattern is rarely needed and easy to misuse (forgetting to call `Start()`, double-starting). `Task.Run` creates and starts in one step.
```

### `Task.Factory.StartNew` — Advanced Control

For scenarios where `Task.Run` isn't enough:

```csharp
// Long-running task — gets a dedicated thread, NOT a pool thread
Task longRunning = Task.Factory.StartNew(() =>
{
    while (true) { ListenForConnections(); }
}, TaskCreationOptions.LongRunning);

// Custom scheduler
Task.Factory.StartNew(() => DoWork(),
    CancellationToken.None,
    TaskCreationOptions.None,
    myCustomScheduler);
```

| | `Task.Run` | `Task.Factory.StartNew` |
|---|---|---|
| Thread | Always pool thread | Pool or dedicated (`LongRunning`) |
| Scheduler | Always `TaskScheduler.Default` | Configurable |
| Unwrapping | Automatically unwraps `Task<Task>` | Does NOT unwrap — can cause bugs |
| Use when | 95% of the time | Need `LongRunning` or custom scheduler |


---

## Task Lifecycle — States

A task goes through defined states tracked by `Task.Status`:

```
             ┌──────────┐
  new Task() │ Created   │  (only if using constructor, not Task.Run)
             └─────┬────┘
                   │ Start() or Task.Run()
             ┌─────▼──────────┐
             │ WaitingToRun    │  queued on thread pool, waiting for a thread
             └─────┬──────────┘
                   │ thread picks it up
             ┌─────▼────┐
             │ Running   │  executing the delegate
             └─────┬────┘
                   │
         ┌─────────┼──────────┐
         │         │          │
   ┌─────▼────┐ ┌──▼──────┐ ┌▼──────────┐
   │RanTo     │ │ Faulted │ │ Canceled  │
   │Completion│ │         │ │           │
   └──────────┘ └─────────┘ └───────────┘
```

| Status | Meaning |
|---|---|
| `Created` | Constructed but not started (only with `new Task()`) |
| `WaitingToRun` | Queued on the scheduler, waiting for a thread |
| `Running` | Currently executing |
| `RanToCompletion` | Finished successfully |
| `Faulted` | Threw an unhandled exception |
| `Canceled` | Was cancelled via `CancellationToken` |

```csharp
Task t = Task.Run(() => Thread.Sleep(2000));

Console.WriteLine(t.Status);       // WaitingToRun or Running
Console.WriteLine(t.IsCompleted);  // false
Console.WriteLine(t.IsFaulted);    // false
Console.WriteLine(t.IsCanceled);   // false

await t;

Console.WriteLine(t.Status);       // RanToCompletion
Console.WriteLine(t.IsCompleted);  // true
```

```ad-note
`IsCompleted` returns `true` for ALL terminal states — `RanToCompletion`, `Faulted`, AND `Canceled`. A faulted task is "completed" (it's done — it just failed). Check `IsCompletedSuccessfully` (.NET Core 2.0+) for success only.
```


---

## Waiting for Tasks

### `await` — The Modern Way (Non-Blocking)

```csharp
int result = await Task.Run(() => Compute());
// Calling thread is FREE while the task runs
// Execution resumes here when the task completes
```

`await` does NOT block the calling thread. It releases it back to the pool (or the message loop in UI apps) and resumes when the task completes.

### `Task.Wait()` — Blocking Wait

```csharp
Task t = Task.Run(() => DoWork());
t.Wait(); // BLOCKS the calling thread until t finishes
```

### `Task.Result` — Blocking Wait + Get Value

```csharp
Task<int> t = Task.Run(() => Compute());
int result = t.Result; // BLOCKS until done, then returns the value
```

```ad-danger
**Do not use `.Wait()` or `.Result` on thread pool threads.** This blocks a pool thread, which can cause thread pool starvation. Use `await` instead. The only safe place for `.Wait()`/`.Result` is in `Main()` or top-level code that isn't on the pool. See [[The CLR Thread Pool]].
```

### `Task.WaitAll` — Wait for Multiple Tasks

```csharp
Task t1 = Task.Run(() => ProcessA());
Task t2 = Task.Run(() => ProcessB());
Task t3 = Task.Run(() => ProcessC());

Task.WaitAll(t1, t2, t3); // blocks until ALL three are done

// With timeout:
bool allDone = Task.WaitAll(new[] { t1, t2, t3 }, TimeSpan.FromSeconds(5));
```

### `Task.WaitAny` — Wait for the First to Finish

```csharp
int completedIndex = Task.WaitAny(t1, t2, t3); // blocks until ONE finishes
Console.WriteLine($"Task {completedIndex} finished first");
```

### `Task.WhenAll` — Async Wait for Multiple Tasks (Non-Blocking)

```csharp
Task<int> t1 = Task.Run(() => ComputeA());
Task<int> t2 = Task.Run(() => ComputeB());

int[] results = await Task.WhenAll(t1, t2); // non-blocking, returns all results
Console.WriteLine($"A={results[0]}, B={results[1]}");
```

### `Task.WhenAny` — Async Wait for First to Finish

```csharp
Task<string> fastest = await Task.WhenAny(
    FetchFromServerA(),
    FetchFromServerB(),
    FetchFromServerC()
);
string result = await fastest; // get the winner's result
```

### Comparison

| Method | Blocking? | Returns when | Use case |
|---|---|---|---|
| `await task` | No | Task completes | Default — almost always use this |
| `task.Wait()` | **Yes** | Task completes | Top-level sync code only |
| `task.Result` | **Yes** | Task completes | Top-level sync code only |
| `Task.WaitAll` | **Yes** | All complete | Sync code, need all results |
| `Task.WaitAny` | **Yes** | First completes | Sync code, need fastest |
| `Task.WhenAll` | No | All complete | Async code, need all results |
| `Task.WhenAny` | No | First completes | Async code, racing/timeout |


---

## Exception Handling

### With `await` — Exceptions Propagate Naturally

```csharp
try
{
    await Task.Run(() => throw new InvalidOperationException("broke"));
}
catch (InvalidOperationException ex)
{
    Console.WriteLine(ex.Message); // "broke" — natural exception handling
}
```

`await` unwraps the `AggregateException` and throws the inner exception directly. This is why `await` is preferred.

### With `.Wait()` or `.Result` — Wrapped in `AggregateException`

```csharp
Task t = Task.Run(() => throw new InvalidOperationException("broke"));

try
{
    t.Wait(); // or t.Result
}
catch (AggregateException ex)
{
    // Must unwrap
    foreach (var inner in ex.InnerExceptions)
        Console.WriteLine(inner.Message);
}
```

### Multiple Exceptions from `Task.WhenAll`

```csharp
Task t1 = Task.Run(() => throw new IOException("file error"));
Task t2 = Task.Run(() => throw new TimeoutException("too slow"));

try
{
    await Task.WhenAll(t1, t2);
}
catch (Exception ex)
{
    // 'await' only throws the FIRST exception
    Console.WriteLine(ex.Message); // "file error"

    // To see ALL exceptions, inspect the tasks directly:
    var allExceptions = Task.WhenAll(t1, t2).Exception; // AggregateException
}
```

### Fire-and-Forget — Unobserved Exceptions

If a faulted task is never awaited, waited on, or has its exception examined, the exception is **unobserved**:

```csharp
Task.Run(() => throw new Exception("nobody sees this"));
// In .NET 4.0: this would crash the process when GC collected the task
// In .NET 4.5+: unobserved exceptions are SWALLOWED by default (logged via event)
```

You can catch these via:

```csharp
TaskScheduler.UnobservedTaskException += (sender, args) =>
{
    Console.WriteLine($"Unobserved: {args.Exception.Message}");
    args.SetObserved(); // prevent it from being rethrown
};
```

```ad-warning
Just because unobserved exceptions don't crash the process in .NET 4.5+ doesn't mean you should ignore them. Always `await` your tasks or handle their exceptions.
```


---

## Cancellation

Tasks support **cooperative cancellation** via `CancellationToken`. The task checks the token periodically and stops voluntarily:

```csharp
var cts = new CancellationTokenSource();

Task work = Task.Run(() =>
{
    for (int i = 0; i < 1_000_000; i++)
    {
        cts.Token.ThrowIfCancellationRequested(); // check periodically
        DoWork(i);
    }
}, cts.Token); // pass token to Task.Run too — lets it cancel before starting

// Cancel after 2 seconds
cts.CancelAfter(TimeSpan.FromSeconds(2));

try
{
    await work;
}
catch (OperationCanceledException)
{
    Console.WriteLine("Task was cancelled");
    Console.WriteLine(work.Status); // Canceled
}
```

### How Cancellation Works

1. `CancellationTokenSource` — the **trigger**. Call `Cancel()` or `CancelAfter()`.
2. `CancellationToken` — the **signal**. Passed into the task. The task checks `ThrowIfCancellationRequested()`.
3. When the token is triggered, the task throws `OperationCanceledException` and transitions to `Canceled` state.

```
CancellationTokenSource ──Cancel()──► CancellationToken
                                           │
                                    ThrowIfCancellationRequested()
                                           │
                                    OperationCanceledException
                                           │
                                    Task.Status = Canceled
```

```ad-note
Cancellation is **cooperative** — the task must actively check the token. If your task calls a long-running method that doesn't check the token, cancellation has no effect until that method returns. Pass the token to inner methods that support it (e.g., `await Task.Delay(5000, token)`).
```


---

## Continuations — Chaining Work After a Task

### `await` — Sequential Chaining (Preferred)

```csharp
string data = await DownloadDataAsync();
var parsed = await ParseAsync(data);
await SaveAsync(parsed);
```

### `ContinueWith` — Callback-Based Chaining

Before `async/await`, `ContinueWith` was the way to chain tasks:

```csharp
Task.Run(() => DownloadData())
    .ContinueWith(prev => Parse(prev.Result))
    .ContinueWith(prev => Save(prev.Result))
    .ContinueWith(prev =>
    {
        if (prev.IsFaulted)
            Console.WriteLine($"Error: {prev.Exception?.InnerException?.Message}");
        else
            Console.WriteLine("All done");
    });
```

| | `await` | `ContinueWith` |
|---|---|---|
| Readability | Sequential, natural flow | Nested callbacks, hard to follow |
| Exception handling | `try/catch` | Must check `prev.IsFaulted` |
| Context capture | Automatic (`SynchronizationContext`) | Manual via `TaskScheduler` |
| Use when | Almost always | Library code, advanced scheduling |

```ad-note
Prefer `await` over `ContinueWith` in application code. `ContinueWith` has many subtle gotchas (default scheduler, no automatic unwrapping, exception handling). It still exists for advanced scenarios and library internals.
```


---

## Creating Completed Tasks

Sometimes you need to return a `Task` that's already done — for interface implementations, caching, or synchronous fast-paths:

```csharp
// Already-completed task with a value
Task<int> cached = Task.FromResult(42);

// Already-completed task (no value)
Task done = Task.CompletedTask;

// Already-cancelled task
Task cancelled = Task.FromCanceled(new CancellationToken(true));

// Already-faulted task
Task faulted = Task.FromException(new IOException("failed"));
```

These are useful when implementing an interface that returns `Task<T>` but you already have the answer:

```csharp
public Task<User> GetUserAsync(int id)
{
    if (_cache.TryGetValue(id, out User user))
        return Task.FromResult(user); // no async needed — return immediately

    return FetchFromDatabaseAsync(id);
}
```


---

## `Task.Delay` — Async Sleep

```csharp
// DON'T do this on pool threads — blocks the thread
Thread.Sleep(2000);

// DO this — releases the thread for 2 seconds, then resumes
await Task.Delay(2000);

// With cancellation
await Task.Delay(5000, cancellationToken);
```

`Task.Delay` is the async equivalent of `Thread.Sleep`. It does NOT block a thread — it sets a timer and the thread returns to the pool. When the timer fires, the continuation is scheduled on a pool thread.


---

## `Task.Yield` — Let Other Work Run

```csharp
await Task.Yield();
```

Forces the current async method to yield control back to the scheduler, allowing other queued work to run. Useful in long-running loops to prevent starvation:

```csharp
async Task ProcessManyItems(IEnumerable<Item> items)
{
    int count = 0;
    foreach (var item in items)
    {
        Process(item);
        if (++count % 100 == 0)
            await Task.Yield(); // let other tasks run every 100 items
    }
}
```


---

## `ValueTask<T>` — Lightweight Task for Hot Paths

`Task<T>` allocates an object on the heap every time. For methods that **usually complete synchronously** (cache hits, buffered reads), this allocation is wasteful:

```csharp
// Allocates a Task<int> on the heap EVERY call, even for cache hits
public async Task<int> GetValueAsync(string key)
{
    if (_cache.TryGetValue(key, out int val))
        return val; // still allocates a Task<int> wrapper
    return await FetchFromDbAsync(key);
}

// ValueTask avoids allocation for the synchronous path
public ValueTask<int> GetValueAsync(string key)
{
    if (_cache.TryGetValue(key, out int val))
        return new ValueTask<int>(val); // no heap allocation
    return new ValueTask<int>(FetchFromDbAsync(key));
}
```

| | `Task<T>` | `ValueTask<T>` |
|---|---|---|
| Heap allocation | Always | Only when truly async |
| Can `await` multiple times | Yes | **No** — can only be awaited once |
| Can call `.Result` after completion | Yes | **No** — use once, discard |
| Use when | General-purpose | Hot paths that often complete synchronously |

```ad-warning
`ValueTask<T>` has restrictions: you can only `await` it **once**, and you cannot cache or reuse it. If you need to await the same result multiple times, use `.AsTask()` to convert it to a regular `Task<T>`.
```


---

## Summary — Task API at a Glance

| Method | What it does |
|---|---|
| `Task.Run(() => ...)` | Queue CPU work on the pool |
| `Task.Factory.StartNew(..., LongRunning)` | Dedicated thread outside the pool |
| `await task` | Non-blocking wait |
| `task.Wait()` / `task.Result` | Blocking wait (avoid on pool threads) |
| `Task.WhenAll(t1, t2)` | Async wait for all |
| `Task.WhenAny(t1, t2)` | Async wait for first |
| `Task.Delay(ms)` | Async sleep (no thread blocked) |
| `Task.Yield()` | Yield to other queued work |
| `Task.FromResult(value)` | Already-completed task |
| `Task.CompletedTask` | Already-completed void task |
| `ContinueWith(...)` | Chain callbacks (prefer `await`) |

See [[Data Parallelism with the Parallel Class]] for data-parallel patterns using `Parallel.For`/`ForEach`. See [[The CLR Thread Pool]] for how tasks are scheduled on the pool.
