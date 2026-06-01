---
tags:
 - csharp
 - threading
 - tpl
---

## Two Kinds of Parallelism

The Task Parallel Library supports two fundamentally different styles of parallelism. Understanding which one you need — and when to combine them — is the key to writing effective parallel code.

```
                        Parallelism
                            │
              ┌─────────────┴─────────────┐
              │                           │
       Data Parallelism            Task Parallelism
              │                           │
    Same operation on              Different operations
    many data items                at the same time
              │                           │
    Parallel.For                   Parallel.Invoke
    Parallel.ForEach               Task.Run + WhenAll
    PLINQ                          Manual Task creation
```

---

## Data Parallelism

Data parallelism means taking a **single operation** and applying it to **many data items concurrently**. The runtime automatically partitions the collection across threads so each core processes a slice of the data.

```
Input:  [A] [B] [C] [D] [E] [F] [G] [H]
         │   │   │   │   │   │   │   │
         └───┴───┘   └───┴───┘   └───┘
         Thread 1    Thread 2    Thread 3
             │           │           │
         Transform   Transform   Transform   ← same operation
             │           │           │
         [A'] [B']   [C'] [D']   [E'] [F'] [G'] [H']
```

The defining characteristic: **every thread runs the same code**, just on different data.

### APIs

| API | Use When |
|---|---|
| `Parallel.For` | Index-based iteration over a range |
| `Parallel.ForEach` | Iterating an `IEnumerable<T>` |
| PLINQ (`.AsParallel()`) | Querying with LINQ — the runtime parallelizes the pipeline |

```csharp
// Data parallelism: same Resize operation on 10,000 images
Parallel.ForEach(images, image =>
{
    Resize(image, 200, 200);  // identical work, different data
});
```

### What Scales

Data parallelism scales with the **size of the data**. Double the items, double the potential speedup (up to the number of cores). Ten images won't saturate an 8-core machine, but 10,000 will.

```ad-note
For a deep dive on `Parallel.For`, `Parallel.ForEach`, partitioning strategies, thread-local accumulation, and stopping/breaking, see [[Data Parallelism with the Parallel Class]].
```


---

## Task Parallelism

Task parallelism means running **different, independent operations** at the same time. Each unit of work does something fundamentally different — they just happen to be independent enough to overlap.

```
         ┌──────────────────┐
         │   Parallel.Invoke │
         │   or Task.WhenAll │
         └────────┬─────────┘
                  │
    ┌─────────────┼─────────────┐
    │             │             │
 Thread 1      Thread 2      Thread 3
    │             │             │
 Compress      Encrypt       Upload       ← different operations
 data          metadata      thumbnail
    │             │             │
    └─────────────┼─────────────┘
                  │
              All done
```

The defining characteristic: **each thread runs different code**, potentially on different data.

### APIs

| API | Use When |
|---|---|
| `Parallel.Invoke` | Fixed set of independent actions, all must complete |
| `Task.Run` + `Task.WhenAll` | Need return values, cancellation, or async composition |
| Manual `Task` / `Task<T>` creation | Complex dependency graphs between tasks |

```csharp
// Task parallelism: three completely different jobs running concurrently
Parallel.Invoke(
    () => CompressPayload(data),
    () => EncryptMetadata(headers),
    () => GenerateThumbnail(image)
);
```

Or with `Task.Run` when you need return values:

```csharp
Task<byte[]> compressed = Task.Run(() => Compress(data));
Task<byte[]> encrypted  = Task.Run(() => Encrypt(headers));
Task<Image>  thumbnail  = Task.Run(() => MakeThumbnail(image));

await Task.WhenAll(compressed, encrypted, thumbnail);

// Retrieve results
byte[] compressedData = compressed.Result;
byte[] encryptedData  = encrypted.Result;
Image  thumb          = thumbnail.Result;
```

### What Scales

Task parallelism scales with the **number of independent tasks**. If you have three independent operations and four cores, the fourth core sits idle. You can only go as wide as the number of distinct things to do.

```ad-note
For more on `Task.Run`, `Task<T>`, continuations, `WhenAll`/`WhenAny`, and exception handling with tasks, see [[The Task Class]].
```


---

## Side-by-Side Comparison

| | Data Parallelism | Task Parallelism |
|---|---|---|
| **What's parallel** | Same operation, many items | Different operations, same time |
| **Typical API** | `Parallel.For`, `Parallel.ForEach`, PLINQ | `Parallel.Invoke`, `Task.Run` + `WhenAll` |
| **Scales with** | Size of the data | Number of independent tasks |
| **Partitioning** | Runtime splits collection into chunks | You define the tasks (one delegate per job) |
| **Mental model** | "Divide the data" | "Divide the work" |
| **Thread utilization** | Many threads doing the same thing | Each thread does something unique |
| **Typical bottleneck** | Work-per-item too small (overhead dominates) | Too few independent tasks to fill all cores |
| **Example** | Apply a filter to every pixel in an image | Compress, encrypt, and upload simultaneously |


---

## The Kitchen Analogy

### Data Parallelism — One Recipe, 100 Cakes

A bakery needs 100 identical cakes. One baker would take all day. Instead, they bring in four bakers — each gets 25 cake orders and follows the **same recipe**. Every baker does the same work, just on a different batch.

```
Baker 1:  Cake 1..25    ─┐
Baker 2:  Cake 26..50   ─┤  same recipe
Baker 3:  Cake 51..75   ─┤  different data (cake #)
Baker 4:  Cake 76..100  ─┘
```

More cakes? Add more bakers. The work scales with the **number of cakes**.

### Task Parallelism — Different Jobs, Same Kitchen

A dinner party needs to be ready by 7 PM. One person doing everything sequentially won't make it. Instead:

- Person A chops vegetables
- Person B stirs the sauce
- Person C sets the table
- Person D arranges flowers

```
Person A:  Chop vegetables    ─┐
Person B:  Stir sauce         ─┤  different jobs
Person C:  Set table          ─┤  running simultaneously
Person D:  Arrange flowers    ─┘
```

Each person does a **completely different job**. The parallelism comes from **independence**, not from data volume. You can only go as wide as the number of distinct tasks — if there are only four jobs, a fifth person has nothing to do.


---

## Mixing Both — Staged Pipeline

Real-world systems often combine both styles: **task parallelism between stages** and **data parallelism within each stage**.

```
Stage 1 (Task A)          Stage 2 (Task B)          Stage 3 (Task C)
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│  Download files  │ ──►  │  Process files   │ ──►  │  Upload results  │
│  (data parallel) │      │  (data parallel) │      │  (data parallel) │
│                  │      │                  │      │                  │
│  Thread 1: d[0]  │      │  Thread 1: p[0]  │      │  Thread 1: u[0]  │
│  Thread 2: d[1]  │      │  Thread 2: p[1]  │      │  Thread 2: u[1]  │
│  Thread 3: d[2]  │      │  Thread 3: p[2]  │      │  Thread 3: u[2]  │
└─────────────────┘      └─────────────────┘      └─────────────────┘
         │                         │                         │
         └── task parallelism ─────┴── between stages ───────┘
                  data parallelism within each stage
```

### Example — Download, Process, Upload Pipeline

```csharp
string[] urls = GetUrls();       // 1000 URLs
string downloadDir = @"C:\temp\downloads";
string resultDir   = @"C:\temp\results";

// Stage 1: Download all files (data parallelism within this stage)
Parallel.ForEach(urls, new ParallelOptions { MaxDegreeOfParallelism = 8 }, url =>
{
    DownloadFile(url, downloadDir);
});

// Stage 2: Process all downloaded files (data parallelism within this stage)
string[] files = Directory.GetFiles(downloadDir);
Parallel.ForEach(files, file =>
{
    var result = HeavyComputation(file);
    Save(result, Path.Combine(resultDir, Path.GetFileName(file)));
});

// Stage 3: Upload all results (data parallelism within this stage)
string[] results = Directory.GetFiles(resultDir);
Parallel.ForEach(results, new ParallelOptions { MaxDegreeOfParallelism = 8 }, file =>
{
    UploadResult(file);
});
```

This is the **simplest** form: each stage runs to completion before the next begins. The task parallelism is sequential between stages; the data parallelism is concurrent within each stage.

```ad-warning
The example above waits for ALL downloads before ANY processing begins. In a real pipeline, you'd want Stage 2 to start processing files as soon as Stage 1 produces them — not wait for all 1,000 downloads. This requires a **producer-consumer** pattern.
```

### True Overlapping Pipeline with Producer-Consumer

To overlap stages so download, process, and upload all run **at the same time** on different items, you need a concurrent queue between stages:

```csharp
using System.Collections.Concurrent;
using System.Threading.Channels;

// Modern approach: use Channels (preferred over BlockingCollection)
var downloadedFiles = Channel.CreateBounded<string>(capacity: 20);
var processedFiles  = Channel.CreateBounded<string>(capacity: 20);

// Stage 1: Download (producer) — task parallelism: this is one "task"
Task downloader = Task.Run(async () =>
{
    await Parallel.ForEachAsync(urls, 
        new ParallelOptions { MaxDegreeOfParallelism = 8 },
        async (url, ct) =>
        {
            string localPath = await DownloadFileAsync(url);
            await downloadedFiles.Writer.WriteAsync(localPath, ct);
        });
    downloadedFiles.Writer.Complete();
});

// Stage 2: Process (consumer of stage 1, producer for stage 3)
Task processor = Task.Run(async () =>
{
    await foreach (string file in downloadedFiles.Reader.ReadAllAsync())
    {
        string result = HeavyComputation(file);  // CPU-bound
        await processedFiles.Writer.WriteAsync(result);
    }
    processedFiles.Writer.Complete();
});

// Stage 3: Upload (consumer of stage 2)
Task uploader = Task.Run(async () =>
{
    await foreach (string file in processedFiles.Reader.ReadAllAsync())
    {
        await UploadResultAsync(file);
    }
});

// All three stages run concurrently — task parallelism between stages
await Task.WhenAll(downloader, processor, uploader);
```

```
Timeline (overlapping stages):

Time ──────────────────────────────────────────────────►

Stage 1:  [download 0] [download 1] [download 2] [download 3] ...
                  │            │            │
Stage 2:         [process 0] [process 1] [process 2] ...
                        │            │
Stage 3:               [upload 0]  [upload 1] ...
```

```ad-note
`Channel<T>` (from `System.Threading.Channels`) is the modern replacement for `BlockingCollection<T>`. It supports async producers and consumers, bounded capacity with back-pressure, and doesn't block threads while waiting. Use `BlockingCollection<T>` only if you're stuck on synchronous code or older .NET versions.
```


---

## How to Choose

```
Do you have MANY items that need the SAME operation?
    │
    ├── YES → Data Parallelism
    │         Parallel.For / ForEach / PLINQ
    │
    └── NO
         │
         Do you have SEVERAL INDEPENDENT operations?
              │
              ├── YES → Task Parallelism
              │         Parallel.Invoke / Task.Run + WhenAll
              │
              └── BOTH?
                   │
                   └── Mix them:
                       Task parallelism between stages
                       Data parallelism within each stage
```

### Quick Decision Table

| Situation | Style | API |
|---|---|---|
| Transform every row in a dataset | Data | `Parallel.ForEach` |
| Resize 10,000 images | Data | `Parallel.ForEach` |
| LINQ query on a large collection | Data | `.AsParallel()` (PLINQ) |
| Compress + encrypt + hash a single file simultaneously | Task | `Parallel.Invoke` |
| Hit 3 independent APIs and combine results | Task | `Task.WhenAll` |
| ETL pipeline: extract, transform, load | Both | Tasks for stages, data parallelism within |
| Render different sections of a report at the same time | Task | `Parallel.Invoke` or `Task.Run` |
| Apply the same filter to every pixel in a bitmap | Data | `Parallel.For` |

```ad-warning
Don't force data parallelism when you actually have task parallelism, or vice versa. If you have three independent API calls, wrapping them in `Parallel.ForEach(new[] { api1, api2, api3 }, ...)` technically works but obscures intent. Use `Parallel.Invoke` or `Task.WhenAll` — it communicates "these are different jobs" rather than "same job, different data."
```


---

## Common Pitfalls

### 1. Using Data Parallelism with Too Few Items

```csharp
// Overkill — 3 items don't benefit from parallel overhead
Parallel.ForEach(new[] { fileA, fileB, fileC }, f => Process(f));

// Better — this IS task parallelism, express it as such
Parallel.Invoke(
    () => Process(fileA),
    () => Process(fileB),
    () => Process(fileC)
);

// Even better if they're truly independent and you need flexibility
await Task.WhenAll(
    Task.Run(() => Process(fileA)),
    Task.Run(() => Process(fileB)),
    Task.Run(() => Process(fileC))
);
```

```ad-note
With only a handful of items, both `Parallel.ForEach` and `Parallel.Invoke` will produce the same runtime behavior. The difference is **communicating intent** to readers of your code: are these the same operation on different data, or different operations that happen to look similar?
```

### 2. Assuming Parallel Always Means Faster

```csharp
// Parallel overhead can make this SLOWER than a plain for loop
Parallel.For(0, 100, i =>
{
    result[i] = i * 2;  // trivial work — overhead of partitioning exceeds the computation
});
```

The parallel infrastructure (partitioning, synchronization, thread pool scheduling) has a fixed cost. If the work-per-item is tiny, that cost dominates and you get a slowdown, not a speedup.

### 3. Accidentally Sequentializing Task Parallelism

```csharp
// WRONG — these run one at a time (await suspends before starting the next)
var a = await Task.Run(() => CompressData());
var b = await Task.Run(() => EncryptData());
var c = await Task.Run(() => GenerateHash());

// RIGHT — start all three, THEN await them together
var taskA = Task.Run(() => CompressData());
var taskB = Task.Run(() => EncryptData());
var taskC = Task.Run(() => GenerateHash());
await Task.WhenAll(taskA, taskB, taskC);
```

```ad-warning
A common mistake with `async`/`await`: if you `await` each task immediately, you serialize them. Start all tasks first, store the `Task` objects, then `await Task.WhenAll(...)` to get true concurrency.
```

### 4. Mixing Up I/O and CPU Parallelism

Data parallelism with `Parallel.For`/`ForEach` is for **CPU-bound** work. If your loop body is mostly waiting on I/O (HTTP calls, file reads, database queries), you're blocking pool threads for no reason. Use `async`/`await` with `Task.WhenAll` for I/O, and `Parallel.*` for CPU.

See the "When to Use" table in [[Data Parallelism with the Parallel Class]] for a detailed breakdown.


---

## Summary

| | Data Parallelism | Task Parallelism |
|---|---|---|
| **Core idea** | Same work, split the data | Different work, run simultaneously |
| **Think** | "Divide the data" | "Divide the jobs" |
| **Scales with** | Collection size | Number of independent operations |
| **Key APIs** | `Parallel.For`, `ForEach`, PLINQ | `Parallel.Invoke`, `Task.Run` + `WhenAll` |
| **Real-world mix** | Use within pipeline stages | Use between pipeline stages |

Most non-trivial systems use **both**: task parallelism to structure independent stages, and data parallelism to process collections within each stage. The choice is not either/or — it's knowing which lens to apply at each level of your design.
