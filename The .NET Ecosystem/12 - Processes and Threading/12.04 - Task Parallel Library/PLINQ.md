---
tags:
 - csharp
 - threading
 - tpl
 - linq
---

## What Is PLINQ?

**PLINQ** (Parallel LINQ) is the parallel version of **LINQ to Objects**. It lives in the `System.Linq` namespace — you call `.AsParallel()` on any `IEnumerable<T>` and the rest of your LINQ query runs in parallel across multiple threads.

```csharp
using System.Linq;
```

Under the hood, PLINQ does three things:

1. **Partitions** the source data into chunks
2. **Executes** the query operators on each chunk using thread pool threads
3. **Merges** the results from all partitions back into a single output sequence

The beauty of PLINQ is that it keeps the **declarative, composable** nature of LINQ while automatically parallelizing the execution. You describe *what* you want (filter, transform, aggregate), and PLINQ decides *how* to distribute it across cores.

```
LINQ to Objects:     source → [operator] → [operator] → [operator] → result
                                  ↓ single thread ↓

PLINQ:               source → [partition] → [operator] → [operator] → [merge] → result
                                  ↓ multiple threads ↓
```

```ad-note
PLINQ only parallelizes **LINQ to Objects** queries — queries over in-memory `IEnumerable<T>` collections. It does NOT parallelize LINQ to SQL, Entity Framework queries, or LINQ to XML queries that hit external data sources. Those have their own execution engines.
```


---

## Basic Usage

### Converting to Parallel — `.AsParallel()`

`.AsParallel()` converts an `IEnumerable<T>` into a `ParallelQuery<T>`. After that, every standard LINQ operator (`Where`, `Select`, `OrderBy`, `GroupBy`, `Aggregate`, etc.) runs in parallel:

```csharp
// Sequential LINQ
var results = data
    .Where(x => Predicate(x))
    .Select(x => Transform(x))
    .ToList();

// PLINQ — same query, parallel execution
var results = data
    .AsParallel()
    .Where(x => Predicate(x))
    .Select(x => Transform(x))
    .ToList();
```

That's it. One method call converts the entire pipeline to parallel execution. The query operators themselves are identical — `Where`, `Select`, `OrderBy` — they just run on multiple threads now.

### Going Back to Sequential — `.AsSequential()`

You can convert a `ParallelQuery<T>` back to a sequential `IEnumerable<T>` at any point in the chain:

```csharp
var results = data
    .AsParallel()
    .Where(x => ExpensivePredicate(x))   // parallel
    .AsSequential()                       // back to sequential from here
    .Take(10)                             // sequential — Take with ordering is tricky in parallel
    .ToList();
```

This is useful when some operators are expensive enough to benefit from parallelism, but downstream operators need sequential semantics (like `Take`, `SkipWhile`, or operators with side effects).

### Anatomy of a PLINQ Query

```
data.AsParallel()                     ← convert IEnumerable<T> to ParallelQuery<T>
    .WithDegreeOfParallelism(4)       ← optional: control thread count
    .AsOrdered()                      ← optional: preserve input order
    .Where(x => x.IsValid)           ← filter (runs in parallel)
    .Select(x => Transform(x))       ← projection (runs in parallel)
    .ToList();                        ← force execution and merge results
```

```ad-note
PLINQ queries are **lazy** just like regular LINQ. The parallel execution doesn't start until you enumerate the results (via `ToList()`, `foreach`, `ToArray()`, `Count()`, etc.). The `.AsParallel()` call just sets up the parallel pipeline — it doesn't execute anything.
```


---

## How PLINQ Works Internally

PLINQ's execution model has three distinct phases: **partition**, **execute**, and **merge**.

### Phase 1: Partitioning

PLINQ splits the source `IEnumerable<T>` into partitions — one per thread that will participate in execution. The partitioning strategy depends on the source type:

| Source type | Partitioning strategy | How it works |
|---|---|---|
| Arrays, `List<T>` | **Range partitioning** | Divides by index ranges — fast, no contention |
| `IEnumerable<T>` (non-indexed) | **Chunk partitioning** | Pulls items from enumerator in small batches under a lock |
| Custom `Partitioner<T>` | **User-defined** | You control the split |

**Range partitioning** is faster because it can divide the data without any synchronization — each thread gets its index range and reads directly from the array.

**Chunk partitioning** requires a lock around the enumerator (since `IEnumerator<T>` is not thread-safe), so threads pull batches and process them, then come back for more. The chunk sizes start small and grow, balancing overhead against load-balancing.

### Phase 2: Parallel Execution

Each partition is processed by a separate thread pool thread. The LINQ operators in the chain execute independently on each partition:

```
Source: [0] [1] [2] [3] [4] [5] [6] [7] [8] [9] [10] [11]

Phase 1 — Partition:
  Partition A: [0] [1] [2] [3]
  Partition B: [4] [5] [6] [7]
  Partition C: [8] [9] [10] [11]

Phase 2 — Execute (in parallel):
  Thread 1 (Partition A): Where → Select → [0'] [2']
  Thread 2 (Partition B): Where → Select → [5'] [7']
  Thread 3 (Partition C): Where → Select → [9'] [11']

Phase 3 — Merge:
  Output: [0'] [2'] [5'] [7'] [9'] [11']
          (order depends on merge strategy — see Merge Options)
```

### Phase 3: Merging

The results from all partitions are combined into a single output sequence. How this merge happens depends on the **merge options** (covered in detail below). By default, PLINQ uses auto-buffered merging — results are gathered in small batches for a balance between latency and throughput.

### Full Flow Diagram

```
              ┌─────────────────────┐
              │   IEnumerable<T>    │
              │   (source data)     │
              └─────────┬───────────┘
                        │
                  .AsParallel()
                        │
              ┌─────────▼───────────┐
              │    Partitioner       │
              │  Splits data into    │
              │  N partitions        │
              └─────────┬───────────┘
                        │
          ┌─────────────┼─────────────┐
          │             │             │
    ┌─────▼─────┐ ┌─────▼─────┐ ┌─────▼─────┐
    │ Thread 1  │ │ Thread 2  │ │ Thread 3  │
    │           │ │           │ │           │
    │  Where    │ │  Where    │ │  Where    │
    │  Select   │ │  Select   │ │  Select   │
    │  ...      │ │  ...      │ │  ...      │
    └─────┬─────┘ └─────┬─────┘ └─────┬─────┘
          │             │             │
          └─────────────┼─────────────┘
                        │
              ┌─────────▼───────────┐
              │   Merge Results     │
              │   into single       │
              │   output sequence   │
              └─────────┬───────────┘
                        │
              ┌─────────▼───────────┐
              │   Final result      │
              │   IEnumerable<T>    │
              └─────────────────────┘
```


---

## Ordering

### Default: No Order Preservation

By default, PLINQ **does not preserve the order** of the source sequence. This is a deliberate design choice — maintaining order requires extra synchronization, which slows things down. If you don't need results in the original order, the default gives the best performance.

```csharp
int[] numbers = Enumerable.Range(0, 20).ToArray();

var results = numbers.AsParallel()
    .Where(n => n % 2 == 0)
    .ToArray();

// results might be: [0, 2, 8, 10, 4, 6, 12, 14, 16, 18]
// NOT guaranteed to be: [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```

### `.AsOrdered()` — Preserve Source Order

When the order of results must match the order of the source, use `.AsOrdered()`:

```csharp
var results = numbers.AsParallel()
    .AsOrdered()
    .Where(n => n % 2 == 0)
    .ToArray();

// results WILL be: [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```

`AsOrdered()` tells PLINQ to track which partition each result came from and its original position, then reassemble them in the correct order during the merge phase. This tracking adds overhead — the threads still run in parallel, but the merge step must buffer and reorder.

### `.AsUnordered()` — Opt Back Out of Ordering

If you called `.AsOrdered()` but a later part of the pipeline doesn't need ordering, you can opt back out:

```csharp
var results = data.AsParallel()
    .AsOrdered()
    .Where(x => x.IsValid)        // order preserved here
    .AsUnordered()
    .Select(x => ExpensiveWork(x)) // order no longer tracked — faster
    .ToList();
```

### When Order Matters vs When It Doesn't

| Scenario | Order matters? | Use |
|---|---|---|
| Displaying results to a user in a specific sequence | Yes | `.AsOrdered()` |
| Aggregating (sum, count, min, max) | No | Default (unordered) |
| Writing results to a file in input order | Yes | `.AsOrdered()` |
| Checking if any item matches a condition | No | Default |
| Generating a sorted report | Depends — if using `OrderBy`, PLINQ handles it | `.OrderBy()` implicitly orders |

```ad-warning
`.AsOrdered()` preserves the **source order** — the order items appeared in the input. It does NOT sort them. If you need sorted results, use `.OrderBy()` (which forces `FullyBuffered` merge and produces sorted output regardless of `AsOrdered`).
```


---

## Degree of Parallelism

### `.WithDegreeOfParallelism(n)` — Control Max Threads

By default, PLINQ uses up to `Environment.ProcessorCount` threads. You can override this:

```csharp
var results = data.AsParallel()
    .WithDegreeOfParallelism(4)
    .Select(x => HeavyComputation(x))
    .ToList();
```

| Value | Meaning |
|---|---|
| `1` | Sequential execution on one thread — useful for debugging |
| `2..N` | At most N threads participate |
| `Environment.ProcessorCount` | Default — use all available cores |
| Max: `512` | Hard upper limit in the framework |

### When to Limit Parallelism

Most of the time, the default is fine. But there are scenarios where limiting threads makes sense:

```csharp
// Example: querying a database that can handle at most 10 concurrent connections
var results = customerIds.AsParallel()
    .WithDegreeOfParallelism(10)
    .Select(id => FetchCustomerFromDb(id))
    .ToList();
```

- **Resource-constrained targets**: Database connections, file handles, API rate limits
- **Memory-intensive operations**: Each thread's work requires a large temporary buffer
- **Shared hardware**: Running on a server where you don't want to consume all cores

```ad-note
Setting degree of parallelism to a number **higher** than `Environment.ProcessorCount` can make sense for I/O-bound PLINQ queries where threads spend most of their time waiting. But as a general rule, **prefer `async/await` for I/O-bound work** rather than PLINQ with inflated thread counts.
```


---

## Cancellation

### `.WithCancellation(token)` — Cancel a Running PLINQ Query

PLINQ supports cooperative cancellation via `CancellationToken`, just like the rest of TPL:

```csharp
var cts = new CancellationTokenSource();

// Cancel after 5 seconds
cts.CancelAfter(TimeSpan.FromSeconds(5));

try
{
    var results = data.AsParallel()
        .WithCancellation(cts.Token)
        .Where(x => ExpensivePredicate(x))
        .Select(x => ExpensiveTransform(x))
        .ToList();
}
catch (OperationCanceledException)
{
    Console.WriteLine("PLINQ query was cancelled.");
}
```

When cancelled:
1. PLINQ checks the token periodically between processing items
2. Threads that are mid-computation finish their current item
3. No new items are picked up
4. An `OperationCanceledException` is thrown on the thread that called `ToList()` (or whatever triggered enumeration)

```ad-warning
PLINQ checks the cancellation token at partition boundaries, not after every single element. If each item takes a long time to process, there will be a delay between requesting cancellation and the query actually stopping. For fine-grained cancellation within a long-running element, check the token manually inside your delegate.
```


---

## Merge Options — `WithMergeOptions`

**Merge options** control how PLINQ delivers results from the parallel threads to the consuming thread. This affects **latency** (how soon you see the first result) vs **throughput** (total time to get all results).

```csharp
var results = data.AsParallel()
    .WithMergeOptions(ParallelMergeOptions.NotBuffered)
    .Where(x => Predicate(x))
    .Select(x => Transform(x));
```

### The Three Options

| Merge option | Behavior | First result latency | Total throughput |
|---|---|---|---|
| `NotBuffered` | Each result yielded immediately as it's produced | **Lowest** | Slower (more sync overhead) |
| `AutoBuffered` | Results gathered in small batches, then yielded | Medium | **Balanced** (default) |
| `FullyBuffered` | ALL results computed and buffered before ANY are yielded | **Highest** | Fastest (minimal sync) |

### When to Use Each

**`NotBuffered`** — Use when you want to start processing results immediately, especially if the consumer is a `foreach` loop that does work on each item as it arrives:

```csharp
// Stream results as they become available
var query = data.AsParallel()
    .WithMergeOptions(ParallelMergeOptions.NotBuffered)
    .Where(x => SlowPredicate(x));

foreach (var item in query)
{
    // Start processing items before the query finishes — lowest latency
    Console.WriteLine(item);
}
```

**`AutoBuffered`** (default) — Good for most scenarios. PLINQ accumulates a batch of results, yields them, then accumulates the next batch. Strikes a balance between responsiveness and overhead.

**`FullyBuffered`** — Use when you need ALL results before doing anything (e.g., sorting). Some operators force this automatically:

```csharp
// OrderBy requires FullyBuffered — you can't yield sorted results
// until you've seen ALL of them
var sorted = data.AsParallel()
    .OrderBy(x => x.Key)
    .ToList();
// FullyBuffered is applied automatically because of OrderBy
```

```ad-note
Operators that inherently require seeing all input before producing output — `OrderBy`, `ThenBy`, `Reverse`, `Distinct`, `GroupBy`, `Aggregate` — implicitly force `FullyBuffered` merging regardless of what you pass to `WithMergeOptions`. The option mainly affects streaming operators like `Where` and `Select`.
```

### Visualizing the Difference

```
NotBuffered:
  Thread 1: ─[A]──────[D]──────────►
  Thread 2: ────[B]─────────[E]────►
  Consumer: ─[A][B]───[D]───[E]───►   ← gets each result immediately

AutoBuffered:
  Thread 1: ─[A]──[D]──────────────►
  Thread 2: ────[B]────[E]─────────►
  Consumer: ──────────[A,B,D]──[E]─►   ← gets results in batches

FullyBuffered:
  Thread 1: ─[A]──[D]──────────────►
  Thread 2: ────[B]────[E]─────────►
  Consumer: ────────────────[A,B,D,E]►  ← gets ALL results at once
```


---

## ForAll — Side-Effect Execution

### What `ForAll` Does

`.ForAll(action)` executes an action on each element **in parallel**, without merging results back to a single consumer thread. This makes it faster than using `foreach` on a PLINQ query when you only need side effects, not a result sequence.

```csharp
data.AsParallel()
    .Where(x => x.IsValid)
    .ForAll(x => Process(x));
```

### Why ForAll Is Faster Than foreach

When you use `foreach` on a PLINQ query, the results must be **merged** from all worker threads onto the single thread running the `foreach` loop. `ForAll` skips this merge entirely — each worker thread executes the action directly on its own partition:

```
foreach on PLINQ result:
  Thread 1: Where → Select → ─┐
  Thread 2: Where → Select → ─┤──► [Merge] ──► single thread: foreach { Process }
  Thread 3: Where → Select → ─┘

ForAll:
  Thread 1: Where → Select → Process   ← no merge step
  Thread 2: Where → Select → Process   ← each thread processes its own items
  Thread 3: Where → Select → Process
```

### Thread Safety with ForAll

Because `ForAll` runs on multiple threads, the action **must be thread-safe**:

```csharp
// UNSAFE — race condition on shared list
var output = new List<string>();
data.AsParallel()
    .Where(x => x.IsValid)
    .ForAll(x => output.Add(x.Name)); // List<T> is NOT thread-safe

// SAFE — use a concurrent collection
var output = new ConcurrentBag<string>();
data.AsParallel()
    .Where(x => x.IsValid)
    .ForAll(x => output.Add(x.Name)); // ConcurrentBag<T> IS thread-safe
```

```ad-warning
`ForAll` gives **no ordering guarantees whatsoever** — even with `.AsOrdered()`. The items are processed in whatever order the threads get to them. If you need ordered side effects, use `foreach` on an `.AsOrdered()` PLINQ query instead.
```

```ad-note
`ForAll` returns `void` — you can't chain more LINQ operators after it. It's always the terminal operation in a PLINQ pipeline.
```


---

## Exception Handling — `AggregateException`

PLINQ follows the same exception pattern as [[Data Parallelism with the Parallel Class|Parallel.For/ForEach]]: exceptions from multiple threads are wrapped into a single `AggregateException`.

```csharp
try
{
    var results = data.AsParallel()
        .Select(x =>
        {
            if (x.IsCorrupt)
                throw new InvalidOperationException($"Bad item: {x.Id}");
            return Transform(x);
        })
        .ToList();
}
catch (AggregateException ex)
{
    foreach (Exception inner in ex.InnerExceptions)
    {
        Console.WriteLine($"Error: {inner.Message}");
    }
}
```

### Exception Flow

1. A thread encounters an exception in its delegate
2. That thread stops processing its partition
3. Other threads **may continue** processing for a short time (they don't abort instantly)
4. PLINQ stops scheduling new work
5. Once all active threads finish their current items, the `AggregateException` is thrown on the consuming thread

### Flattening Nested AggregateExceptions

If your delegate itself throws `AggregateException` (e.g., it awaits a task internally), you can end up with nested aggregates. Use `Flatten()` to unwrap them:

```csharp
catch (AggregateException ex)
{
    foreach (Exception inner in ex.Flatten().InnerExceptions)
    {
        Console.WriteLine($"Error: {inner.Message}");
    }
}
```


---

## PLINQ vs `Parallel.ForEach`

Both PLINQ and `Parallel.ForEach` (from [[Data Parallelism with the Parallel Class]]) achieve data parallelism, but they have different strengths:

| | PLINQ | `Parallel.ForEach` |
|---|---|---|
| **Style** | Declarative — query syntax | Imperative — statement body |
| **Returns results** | Yes — produces `IEnumerable<T>` | No — side-effect based |
| **Composable** | Yes — chains with other LINQ operators | No — standalone call |
| **Ordering control** | `.AsOrdered()` / `.AsUnordered()` | No ordering control |
| **Merge options** | `WithMergeOptions` (NotBuffered, AutoBuffered, FullyBuffered) | No merge options |
| **Early termination** | No `Break()`/`Stop()` | `ParallelLoopState.Break()` / `.Stop()` |
| **Execution mode override** | `WithExecutionMode(ForceParallelism)` | Always parallel |
| **Thread-local accumulation** | Use `Aggregate` operator | Thread-local overload with `localInit`/`localFinally` |
| **Cancellation** | `.WithCancellation(token)` | `ParallelOptions.CancellationToken` |
| **When to use** | Querying, filtering, transforming, aggregating data | Side effects: process, save, send, modify each item |

### Rule of Thumb

- **Need a result collection?** Use PLINQ — it returns results naturally
- **Need to perform actions (side effects) on each item?** Use `Parallel.ForEach`
- **Need to compose with other LINQ operators?** Use PLINQ
- **Need Break/Stop semantics?** Use `Parallel.ForEach`

```csharp
// PLINQ — when you want to produce a new collection
var validItems = items.AsParallel()
    .Where(x => x.IsValid)
    .Select(x => new ProcessedItem(x))
    .ToList();

// Parallel.ForEach — when you want to DO something to each item
Parallel.ForEach(items, item =>
{
    SaveToDatabase(item);
});
```


---

## Execution Mode — When PLINQ Falls Back to Sequential

PLINQ is not blindly parallel. Before executing, it analyzes the query and may **silently fall back to sequential execution** if it determines that parallelism won't help or could hurt. This can happen when:

- The query is too simple (overhead would dominate)
- The query contains operators that are inherently sequential
- The source is very small

You can override this with `WithExecutionMode`:

```csharp
// Force parallel execution even if PLINQ thinks sequential would be better
var results = data.AsParallel()
    .WithExecutionMode(ParallelExecutionMode.ForceParallelism)
    .Select(x => Transform(x))
    .ToList();
```

| Mode | Behavior |
|---|---|
| `ParallelExecutionMode.Default` | PLINQ decides — may fall back to sequential |
| `ParallelExecutionMode.ForceParallelism` | Always execute in parallel, even if PLINQ thinks it won't help |

```ad-note
`ForceParallelism` is mostly useful for **benchmarking and testing**. In production, PLINQ's heuristic is usually right — if it thinks sequential is faster, it probably is. The main exception is when your delegate has side effects or latency that PLINQ's static analysis can't account for (e.g., network calls within a `Select`).
```


---

## When NOT to Use PLINQ

PLINQ is powerful but not universally applicable. Avoid it in these situations:

### 1. Trivial/Fast Operations

```csharp
// BAD — the work per item (multiplication) is so fast that
// partitioning + thread scheduling + merging costs more than just doing it sequentially
var doubled = numbers.AsParallel().Select(n => n * 2).ToList();

// GOOD — just use regular LINQ
var doubled = numbers.Select(n => n * 2).ToList();
```

### 2. I/O-Bound Work

PLINQ uses thread pool threads, which **block** while waiting for I/O. This leads to thread pool starvation. Use `async/await` instead:

```csharp
// BAD — blocks pool threads waiting for HTTP responses
var pages = urls.AsParallel()
    .Select(url => httpClient.GetStringAsync(url).Result) // .Result blocks!
    .ToList();

// GOOD — async I/O, no threads wasted
var tasks = urls.Select(url => httpClient.GetStringAsync(url));
var pages = await Task.WhenAll(tasks);
```

### 3. Side Effects That Aren't Thread-Safe

If your `Where` or `Select` delegates mutate shared state without synchronization, PLINQ will introduce race conditions:

```csharp
// DANGEROUS — counter++ is a race condition across threads
int counter = 0;
data.AsParallel()
    .Where(x => { counter++; return x.IsValid; }) // NOT thread-safe
    .ToList();
```

### 4. Queries That Depend on Ordering and Can't Afford AsOrdered Overhead

If your pipeline relies heavily on element order AND the `AsOrdered` overhead negates the parallelism benefit, sequential LINQ is the better choice.

### 5. Very Small Collections

For a handful of items, the overhead of partitioning and thread scheduling exceeds any speedup:

| Collection size | Work per item | Parallel worth it? |
|---|---|---|
| 10 | Trivial | No |
| 100 | Light | Unlikely |
| 1,000 | Moderate | Probably |
| 10,000+ | Moderate to heavy | Yes |
| 100,000+ | Any non-trivial work | Almost certainly yes |

```ad-warning
**PLINQ can silently fall back to sequential** if it determines parallelism won't help. This is usually fine — but if you're not seeing the speedup you expect, check whether PLINQ decided to go sequential. Use `.WithExecutionMode(ParallelExecutionMode.ForceParallelism)` to confirm, then benchmark both.
```


---

## Custom Partitioning

For advanced scenarios, you can supply a custom `Partitioner<T>` to control how PLINQ splits the data:

```csharp
using System.Collections.Concurrent;

// Create a range partitioner with specific chunk sizes
var partitioner = Partitioner.Create(0, data.Length, rangeSize: 500);

var results = partitioner.AsParallel()
    .Select(range =>
    {
        // Process a contiguous range of indices
        int sum = 0;
        for (int i = range.Item1; i < range.Item2; i++)
            sum += data[i];
        return sum;
    })
    .Sum();
```

Use cases for custom partitioning:
- **Cache locality**: Keep nearby elements on the same thread to exploit CPU cache lines
- **Load balancing**: When items have wildly different processing costs, smaller chunks balance better
- **Avoiding lock contention**: Range partitioning on arrays avoids the lock that chunk partitioning needs


---

## Complete Example — Processing a Large Dataset

Here's a practical example that ties together PLINQ features: filtering, transforming, aggregating, and handling errors — processing a large collection of sensor readings.

```csharp
using System;
using System.Collections.Concurrent;
using System.Diagnostics;
using System.Linq;
using System.Threading;

// Simulate a large dataset of sensor readings
var readings = Enumerable.Range(0, 1_000_000)
    .Select(i => new SensorReading
    {
        SensorId = i % 100,
        Timestamp = DateTime.UtcNow.AddSeconds(-i),
        Value = Math.Sin(i * 0.01) * 100 + Random.Shared.NextDouble() * 10,
        IsCalibrated = i % 7 != 0 // every 7th reading is uncalibrated
    })
    .ToArray();

var cts = new CancellationTokenSource();

var sw = Stopwatch.StartNew();

try
{
    // PLINQ pipeline: filter, transform, group, aggregate
    var sensorAverages = readings
        .AsParallel()
        .WithCancellation(cts.Token)
        .WithDegreeOfParallelism(Environment.ProcessorCount)
        .Where(r => r.IsCalibrated)                         // filter out bad readings
        .Where(r => r.Value >= -200 && r.Value <= 200)      // remove outliers
        .Select(r => new                                     // project to anonymous type
        {
            r.SensorId,
            NormalizedValue = Math.Round(r.Value, 2)
        })
        .GroupBy(r => r.SensorId)                            // group by sensor
        .Select(group => new
        {
            SensorId = group.Key,
            Average = group.Average(r => r.NormalizedValue),
            Count = group.Count(),
            Min = group.Min(r => r.NormalizedValue),
            Max = group.Max(r => r.NormalizedValue)
        })
        .OrderBy(s => s.SensorId)                            // sort by sensor ID
        .ToList();

    sw.Stop();

    Console.WriteLine($"Processed {readings.Length:N0} readings in {sw.ElapsedMilliseconds} ms");
    Console.WriteLine($"Found {sensorAverages.Count} sensors\n");

    foreach (var sensor in sensorAverages.Take(5))
    {
        Console.WriteLine(
            $"Sensor {sensor.SensorId,3}: " +
            $"Avg={sensor.Average,8:F2}  " +
            $"Min={sensor.Min,8:F2}  " +
            $"Max={sensor.Max,8:F2}  " +
            $"Count={sensor.Count}");
    }
}
catch (OperationCanceledException)
{
    Console.WriteLine("Query cancelled.");
}
catch (AggregateException ex)
{
    foreach (var inner in ex.Flatten().InnerExceptions)
        Console.WriteLine($"Error: {inner.Message}");
}

record SensorReading
{
    public int SensorId { get; init; }
    public DateTime Timestamp { get; init; }
    public double Value { get; init; }
    public bool IsCalibrated { get; init; }
}
```

### Sequential vs PLINQ Comparison

To see the actual speedup, compare the same pipeline with and without `.AsParallel()`:

```csharp
// Sequential
var sw1 = Stopwatch.StartNew();
var seqResult = readings
    .Where(r => r.IsCalibrated && r.Value >= -200 && r.Value <= 200)
    .Select(r => new { r.SensorId, NormalizedValue = Math.Round(r.Value, 2) })
    .GroupBy(r => r.SensorId)
    .Select(g => new { SensorId = g.Key, Average = g.Average(r => r.NormalizedValue) })
    .OrderBy(s => s.SensorId)
    .ToList();
sw1.Stop();

// PLINQ
var sw2 = Stopwatch.StartNew();
var parResult = readings
    .AsParallel()
    .Where(r => r.IsCalibrated && r.Value >= -200 && r.Value <= 200)
    .Select(r => new { r.SensorId, NormalizedValue = Math.Round(r.Value, 2) })
    .GroupBy(r => r.SensorId)
    .Select(g => new { SensorId = g.Key, Average = g.Average(r => r.NormalizedValue) })
    .OrderBy(s => s.SensorId)
    .ToList();
sw2.Stop();

Console.WriteLine($"Sequential: {sw1.ElapsedMilliseconds} ms");
Console.WriteLine($"PLINQ:      {sw2.ElapsedMilliseconds} ms");
Console.WriteLine($"Speedup:    {(double)sw1.ElapsedMilliseconds / sw2.ElapsedMilliseconds:F1}x");
```

```ad-note
On an 8-core machine with 1 million items and moderate work per item, expect roughly a 3-6x speedup. You typically won't see a perfect 8x because of partitioning overhead, merge costs, and the fact that `GroupBy` and `OrderBy` have inherently sequential bottlenecks (all data must be seen before output is produced).
```


---

## Quick Reference — PLINQ Extension Methods

| Method | Purpose |
|---|---|
| `.AsParallel()` | Convert `IEnumerable<T>` to `ParallelQuery<T>` |
| `.AsSequential()` | Convert back to sequential `IEnumerable<T>` |
| `.AsOrdered()` | Preserve source element order |
| `.AsUnordered()` | Opt out of order preservation |
| `.WithDegreeOfParallelism(n)` | Limit max threads |
| `.WithCancellation(token)` | Support cooperative cancellation |
| `.WithMergeOptions(option)` | Control result buffering strategy |
| `.WithExecutionMode(mode)` | Force parallel or let PLINQ decide |
| `.ForAll(action)` | Execute side effects in parallel (no merge) |
| `.Aggregate(...)` | Thread-safe parallel aggregation |

All standard LINQ operators (`Where`, `Select`, `SelectMany`, `OrderBy`, `GroupBy`, `Join`, `Distinct`, `Union`, `Intersect`, `Except`, `Concat`, `Take`, `Skip`, `Count`, `Sum`, `Average`, `Min`, `Max`, `Aggregate`, etc.) work on `ParallelQuery<T>`.


---

## Key Takeaways

- **PLINQ = LINQ + `.AsParallel()`** — minimal code change for parallel execution
- Ideal for **CPU-bound** data processing on large collections
- **Not for I/O-bound work** — use `async/await` instead
- Default behavior: **unordered**, auto-buffered merge, processor count threads
- Use `.AsOrdered()` when order matters, knowing it adds overhead
- Use `ForAll` for parallel side effects without the merge step
- Exceptions are wrapped in `AggregateException`
- PLINQ may **silently go sequential** — use `ForceParallelism` to override
- **Measure before and after** — not every query benefits from parallelism
- For side-effect-heavy imperative loops, prefer [[Data Parallelism with the Parallel Class|Parallel.ForEach]] instead
- For task-level parallelism (different operations at the same time), see [[The Task Class]] and [[Task Parallelism vs Data Parallelism]]
