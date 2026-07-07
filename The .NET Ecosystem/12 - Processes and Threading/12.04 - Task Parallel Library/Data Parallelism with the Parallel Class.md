---
tags:
 - csharp
 - threading
 - tpl
---

## What Is Data Parallelism?

Data parallelism means performing the **same operation on many items** at the same time by splitting the data across multiple threads. Instead of one thread iterating through 10,000 items sequentially, four threads each handle 2,500 items concurrently.

```
Sequential:     Thread 1 → [item 0] [item 1] [item 2] ... [item 9999]

Data Parallel:  Thread 1 → [item 0..2499]
                Thread 2 → [item 2500..4999]
                Thread 3 → [item 5000..7499]
                Thread 4 → [item 7500..9999]
```

The `System.Threading.Tasks.Parallel` class provides the methods that make this easy: `Parallel.For`, `Parallel.ForEach`, and `Parallel.Invoke`.

```csharp
using System.Threading.Tasks;
```


---

## `Parallel.For` — Parallel Index-Based Loop

`Parallel.For` is the parallel equivalent of a `for` loop. It partitions the iteration range across pool threads:

```csharp
// Sequential
for (int i = 0; i < 10000; i++)
    results[i] = Compute(i);

// Parallel — same logic, multiple threads
Parallel.For(0, 10000, i =>
{
    results[i] = Compute(i);
});
```

### How It Works Internally

1. The range `[0, 10000)` is split into **chunks** (partitions)
2. Each chunk is assigned to a thread pool thread
3. Threads execute their chunks concurrently
4. `Parallel.For` **blocks the calling thread** until all iterations are complete

```
Parallel.For(0, 10000, ...)

Calling thread BLOCKS here
    │
    ├── Pool Thread 1: iterations 0..2499
    ├── Pool Thread 2: iterations 2500..4999
    ├── Pool Thread 3: iterations 5000..7499
    └── Pool Thread 4: iterations 7500..9999
    │
    ▼ all done
Calling thread resumes
```

```ad-note
The partitioning is NOT guaranteed to be even or sequential. The runtime decides how to split based on the number of available cores and current thread pool state. Each thread may also process its chunk in any order. **Do not assume iterations run in order.**
```

### Return Value — `ParallelLoopResult`

`Parallel.For` returns a `ParallelLoopResult` that tells you whether the loop ran to completion:

```csharp
ParallelLoopResult result = Parallel.For(0, 10000, i =>
{
    Process(i);
});

Console.WriteLine($"Completed: {result.IsCompleted}");
// If early-stopped, LowestBreakIteration shows where
Console.WriteLine($"Lowest break: {result.LowestBreakIteration}");
```


---

## `Parallel.ForEach` — Parallel Collection Iteration

`Parallel.ForEach` is the parallel equivalent of `foreach`. It works on any `IEnumerable<T>`:

```csharp
List<string> files = GetFilePaths();

// Sequential
foreach (string file in files)
    ProcessFile(file);

// Parallel
Parallel.ForEach(files, file =>
{
    ProcessFile(file);
});
```

### With Index Access

If you need the index of each item:

```csharp
Parallel.ForEach(files, (file, state, index) =>
{
    Console.WriteLine($"[{index}] Processing {file} on thread {Thread.CurrentThread.ManagedThreadId}");
    ProcessFile(file);
});
```

The three parameters are:
- `file` — the current item
- `state` — a `ParallelLoopState` for controlling the loop (stop/break)
- `index` — the zero-based index of the item


---

## `Parallel.Invoke` — Run Independent Actions Concurrently

When you have a fixed number of independent operations (not a loop), use `Parallel.Invoke`:

```csharp
Parallel.Invoke(
    () => DownloadFile("a.txt"),
    () => DownloadFile("b.txt"),
    () => CompressData(),
    () => GenerateReport()
);
// All four run concurrently. Invoke blocks until ALL are done.
```

Each action runs on a pool thread. `Invoke` blocks the calling thread until every action completes.

```ad-note
`Parallel.Invoke` is for a **fixed set of independent actions**. If you have a collection of items to process, use `Parallel.ForEach` instead. If you need async support, use `Task.WhenAll`.
```


---

## Controlling Parallelism with `ParallelOptions`

All three methods accept a `ParallelOptions` parameter for fine-grained control:

```csharp
var options = new ParallelOptions
{
    MaxDegreeOfParallelism = 4,                        // at most 4 threads
    CancellationToken = cts.Token,                     // cancellation support
    TaskScheduler = TaskScheduler.Default               // which scheduler to use
};

Parallel.ForEach(items, options, item =>
{
    Process(item);
});
```

### `MaxDegreeOfParallelism`

Controls how many threads can execute simultaneously:

| Value | Meaning                                                 |
| ----- | ------------------------------------------------------- |
| `-1`  | No limit — the runtime decides (default)                |
| `1`   | Sequential execution — one thread, useful for debugging | 
| `N`   | At most N threads concurrently                          |

```csharp
// Limit to 4 concurrent threads (e.g., to avoid overwhelming a database)
var options = new ParallelOptions { MaxDegreeOfParallelism = 4 };

Parallel.ForEach(queries, options, query =>
{
    ExecuteQuery(query); // at most 4 concurrent queries
});
```

### `CancellationToken`

Allows cancelling the parallel operation from outside:

```csharp
var cts = new CancellationTokenSource();

// Cancel after 5 seconds
cts.CancelAfter(TimeSpan.FromSeconds(5));

try
{
    Parallel.For(0, 1_000_000, new ParallelOptions { CancellationToken = cts.Token }, i =>
    {
        DoExpensiveWork(i);
    });
}
catch (OperationCanceledException)
{
    Console.WriteLine("Parallel loop was cancelled");
}
```

When cancelled, iterations that haven't started are skipped. Iterations that are already running finish their current work, then the loop stops.


---

## Stopping and Breaking Early

### `ParallelLoopState.Stop()` — Halt Everything

Signals all threads to stop as soon as possible. No new iterations will start. Already-running iterations finish, then the loop returns.

```csharp
Parallel.ForEach(files, (file, state) =>
{
    if (file.Contains("POISON"))
    {
        state.Stop(); // tell all threads to stop
        return;
    }
    ProcessFile(file);
});
```

After `Stop()`:
- `state.IsStopped` is `true` for all threads
- No guarantee that iterations after the "POISON" item won't have started already (they run in parallel)

### `ParallelLoopState.Break()` — Finish Everything Below This Index

`Break()` is softer than `Stop()`. It says: "complete all iterations with an index lower than mine, then stop."

```csharp
Parallel.For(0, 10000, (i, state) =>
{
    if (FoundAnswer(i))
    {
        state.Break(); // finish everything below i, skip everything above
        return;
    }
    Process(i);
});
```

### `Stop` vs `Break`

|                       | `Stop()`                            | `Break()`                                                  |
| --------------------- | ----------------------------------- | ---------------------------------------------------------- |
| Meaning               | "Abort — stop everything"           | "I found something at index N — finish everything below N" |
| New iterations start? | No                                  | Only if index < break point                                |
| Use case              | Error, poison pill, fatal condition | Search — found the answer, no need to go further           |


---

## Partitioning — How Work Gets Divided

The runtime doesn't just assign one iteration per thread. It splits the range into **chunks** to minimize synchronization overhead:

```
Range: [0..10000)    Threads: 4

Chunk-based partitioning:
  Thread 1 takes chunk [0..625)     ← processes 625 items, comes back for more
  Thread 2 takes chunk [625..1250)
  Thread 3 takes chunk [1250..1875)
  Thread 4 takes chunk [1875..2500)
  Thread 1 finishes first → takes chunk [2500..3125)
  ... continues until all chunks are done
```

This is called **dynamic partitioning** — fast threads get more chunks, slow threads get fewer. This naturally load-balances across threads.

For `ForEach` on an `IEnumerable<T>` (not indexed), the runtime uses **buffered partitioning** — it pulls items from the enumerator in small batches and hands each batch to a thread.

```ad-note
You can provide a custom partitioner via `Partitioner.Create` for specialized scenarios (e.g., if items near each other in the array share cache lines and should be processed by the same thread):

```csharp
var partitioner = Partitioner.Create(0, 10000, rangeSize: 100);
Parallel.ForEach(partitioner, range =>
{
    for (int i = range.Item1; i < range.Item2; i++)
        Process(i);
});
```


---

## Exception Handling — `AggregateException`

If one or more iterations throw exceptions, `Parallel.For`/`ForEach` collects them all into an `AggregateException`:

```csharp
try
{
    Parallel.ForEach(items, item =>
    {
        if (item.IsCorrupt)
            throw new InvalidOperationException($"Bad item: {item.Id}");

        Process(item);
    });
}
catch (AggregateException ex)
{
    // Multiple iterations may have thrown — each is an InnerException
    foreach (Exception inner in ex.InnerExceptions)
    {
        Console.WriteLine($"Error: {inner.Message}");
    }
}
```

When an exception is thrown:
1. That iteration terminates
2. Other **already-running** iterations continue to completion (they don't get aborted)
3. **New** iterations may or may not start (the runtime tries to stop soon)
4. After all running iterations finish, the `AggregateException` is thrown on the calling thread

```ad-warning
The parallel loop does NOT stop immediately on the first exception. Other threads that are already running finish their current iteration. If you need immediate stop on error, combine exception handling with `state.Stop()`:

```csharp
Parallel.ForEach(items, (item, state) =>
{
    try
    {
        Process(item);
    }
    catch
    {
        state.Stop(); // signal other threads to stop
        throw;        // still propagate into AggregateException
    }
});
```


---

## Thread Safety Inside Parallel Loops

The body of a `Parallel.For`/`ForEach` runs on **multiple threads simultaneously**. This means you must handle shared state carefully.

### SAFE — Each Iteration Writes to Its Own Index

```csharp
int[] results = new int[10000];

Parallel.For(0, 10000, i =>
{
    results[i] = Compute(i); // each thread writes to a DIFFERENT index — safe
});
```

### UNSAFE — Multiple Iterations Modify Shared State

```csharp
int total = 0;

Parallel.For(0, 10000, i =>
{
    total += values[i]; // RACE CONDITION — multiple threads read-modify-write total
});
```

### FIX — Use `Interlocked` or Thread-Local Accumulation

```csharp
// Option 1: Interlocked (simple but high contention)
int total = 0;
Parallel.For(0, 10000, i =>
{
    Interlocked.Add(ref total, values[i]);
});

// Option 2: Thread-local accumulation (better for performance)
int total = 0;
Parallel.For(
    fromInclusive: 0,
    toExclusive: 10000,
    localInit: () => 0,                        // each thread starts with local sum = 0
    body: (i, state, localSum) =>
    {
        localSum += values[i];                 // accumulate locally — no contention
        return localSum;
    },
    localFinally: localSum =>
    {
        Interlocked.Add(ref total, localSum);  // merge once per thread — minimal contention
    }
);
```

The thread-local overload is the **correct way to aggregate** in parallel loops. Each thread accumulates into its own private variable, then merges once at the end. This avoids the contention of 10,000 `Interlocked.Add` calls.


---

## Thread Affinity and GUI Applications

```ad-danger
Secondary threads (including thread pool threads used by `Parallel`) can **never directly access UI controls**. UI controls have thread affinity — they can only be touched by the thread that created them (the UI thread).

```csharp
// WRONG — crashes with cross-thread exception
Parallel.ForEach(items, item =>
{
    progressBar.Value += 1; // accessing UI from pool thread — ILLEGAL
});

// RIGHT — marshal updates back to the UI thread
Parallel.ForEach(items, item =>
{
    Process(item);
    // Use Invoke/BeginInvoke (WinForms) or Dispatcher (WPF) to update UI
    this.Invoke(() => progressBar.Value += 1);
});
```


---

## When to Use Parallel vs async/await vs Task.Run

| Scenario | Use |
|---|---|
| CPU-bound work over a collection (compute, transform, process) | `Parallel.For` / `Parallel.ForEach` |
| Fixed set of independent CPU-bound actions | `Parallel.Invoke` |
| I/O-bound work (HTTP, files, DB) | `async/await` with `Task.WhenAll` — NOT Parallel |
| Single CPU-bound operation off the UI thread | `Task.Run` |
| Need to stream results as they complete | PLINQ or `Task.WhenAny` loop |

```ad-warning
**Do NOT use `Parallel.ForEach` for I/O-bound work.** It blocks pool threads while waiting for I/O, causing thread pool starvation. Use `async/await` instead:

```csharp
// WRONG — blocks pool threads on I/O
Parallel.ForEach(urls, url =>
{
    var data = httpClient.GetStringAsync(url).Result; // blocks!
});

// RIGHT — async I/O, no threads blocked
var tasks = urls.Select(url => httpClient.GetStringAsync(url));
var results = await Task.WhenAll(tasks);
```


---

## When NOT to Use Parallel

Parallelism has overhead — partitioning, thread coordination, cache effects, synchronization. For some workloads, the overhead exceeds the benefit:

| Situation                                    | Parallel worth it?                     |
| -------------------------------------------- | -------------------------------------- |
| 10 million items, heavy computation per item | Yes — massive speedup                  |
| 10,000 items, moderate work per item         | Probably yes                           |
| 100 items, light work per item               | Probably no — overhead dominates       |
| 10 items, trivial work                       | No — sequential is faster              |
| Items have dependencies on each other        | No — parallelism requires independence |
| Body is mostly I/O                           | No — use async instead                 |

```ad-note
When in doubt, **measure**. Use `Stopwatch` to compare sequential vs parallel. The crossover point depends on work-per-item, number of items, and number of CPU cores.
```


---

## Complete Example — Parallel Image Processing

```csharp
string[] imagePaths = Directory.GetFiles(@"C:\Photos", "*.jpg");

var options = new ParallelOptions { MaxDegreeOfParallelism = Environment.ProcessorCount };

Parallel.ForEach(imagePaths, options, (path, state, index) =>
{
    try
    {
        Console.WriteLine($"[{index}] Processing {Path.GetFileName(path)} " +
                          $"on thread {Thread.CurrentThread.ManagedThreadId}");

        using var image = LoadImage(path);
        var thumbnail = Resize(image, 200, 200);
        Save(thumbnail, Path.Combine(@"C:\Thumbnails", Path.GetFileName(path)));
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error processing {path}: {ex.Message}");
        // Don't rethrow — let other images continue processing
    }
});

Console.WriteLine($"Processed {imagePaths.Length} images.");
```
