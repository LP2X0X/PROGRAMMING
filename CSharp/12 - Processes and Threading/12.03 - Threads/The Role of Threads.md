---
tags:
 - csharp
 - processes
---

## What Is a Thread?

A thread is a **path of execution within a process**. It is the smallest unit of work that the OS scheduler can manage. Every process has at least one thread — the **primary thread** — which is created when the process starts and serves as the application's entry point (typically `Main()`).

```ad-note
Processes that contain only a single primary thread are intrinsically **thread-safe**, because only one thread can access the application's data at any given time.
```

```ad-note
**The same thread can execute work belonging to different logical contexts over its lifetime.** The thread is the active worker. What changes is the work it picks up and the context that work carries.
```


---

## Thread Name

Every thread can be assigned a **human-readable name** via `Thread.Name`. This name has no effect on behavior — it exists purely for **debugging and diagnostics**. It shows up in the Visual Studio Threads window, log output, and exception stack traces.

```csharp
Thread worker = new Thread(DoWork);
worker.Name = "DataProcessor";
worker.Start();

// Inside any thread, you can read its own name:
Console.WriteLine($"Running on: {Thread.CurrentThread.Name}");
```

```ad-warning
`Thread.Name` can only be set **once**. Attempting to set it a second time throws an `InvalidOperationException`. This is by design — once a thread has a name, the runtime and debugger rely on it being stable.
```

By default, threads have no name (`null`). The primary thread's name is also `null` unless you set it explicitly:

```csharp
Thread.CurrentThread.Name = "Main";
```

Thread pool threads (used by `Task.Run`) start unnamed. You can name them when they begin executing your work, but since they're reused, the name sticks from the first assignment and can't be changed.


---

## Thread Priority

Each thread has a `Priority` property that provides a **hint** to the OS scheduler about how much CPU time this thread should receive relative to other threads in the same process.

```csharp
Thread worker = new Thread(CrunchNumbers);
worker.Priority = ThreadPriority.BelowNormal;
worker.Start();
```

### The Five Priority Levels

| Priority | Value | Meaning |
|---|---|---|
| `ThreadPriority.Highest` | 4 | Scheduled before all lower-priority threads |
| `ThreadPriority.AboveNormal` | 3 | More CPU time than normal |
| `ThreadPriority.Normal` | 2 | Default — fair share |
| `ThreadPriority.BelowNormal` | 1 | Less CPU time than normal |
| `ThreadPriority.Lowest` | 0 | Only runs when no higher-priority threads need the CPU |

### How Priority Actually Works

Thread priority does **not** control execution order. A `Highest` priority thread is not guaranteed to run first. What the OS scheduler does is:

1. When multiple threads are **ready to run** and competing for the same CPU core, the higher-priority thread gets the core
2. A lower-priority thread still runs — it just gets fewer time slices
3. The OS can and will **boost** priorities temporarily (e.g., a thread that just received user input gets a brief priority bump)

```ad-danger
Setting a thread to `Highest` can **starve** lower-priority threads — they may barely get any CPU time if the high-priority thread never blocks. Setting a thread to `RealTime` (via process priority, not `ThreadPriority`) can make the **entire system** unresponsive. In practice, most applications should leave priority at `Normal`.
```

### When to Adjust Priority

| Scenario | Priority | Why |
|---|---|---|
| Background file indexing | `BelowNormal` | Don't compete with the user's active work |
| UI thread | `Normal` (default) | Default is fine — the OS already boosts it on input |
| Time-critical audio processing | `AboveNormal` | Needs consistent scheduling to avoid glitches |
| Almost never | `Highest` | Risk of starving other threads |


---

## Primary Thread vs. Worker Threads

- The **primary thread** is the first thread created by the process. In a GUI application, this is the **UI thread** — it owns the message loop and handles user interaction.
- **Worker threads** (also called secondary threads) are additional threads spawned by the primary thread to perform concurrent work.

```
┌──────────────────────────────────────────┐
│              Process                     │
│                                          │
│  ┌──────────────┐                        │
│  │ Primary       │──── Entry point       │
│  │ Thread        │     (Main method)     │
│  └──────┬───────┘                        │
│         │ spawns                         │
│         ├──────────────┐                 │
│         │              │                 │
│  ┌──────▼─────┐  ┌────▼───────┐         │
│  │ Worker      │  │ Worker      │        │
│  │ Thread 1    │  │ Thread 2    │        │
│  └────────────┘  └─────────────┘         │
│                                          │
│  ┌──────────────────────────────────┐    │
│  │ Shared Memory (Heap, Static)     │    │
│  │ ← All threads read/write here   │    │
│  └──────────────────────────────────┘    │
└──────────────────────────────────────────┘
```


---

## Foreground Threads vs Background Threads

This is a **separate classification** from primary/worker. Primary vs worker describes **who created the thread**. Foreground vs background describes **whether the thread keeps the process alive**.

- **Foreground threads** keep the process alive. The process **will not exit** until every foreground thread has finished, even if `Main()` has returned.
- **Background threads** are killed automatically the moment all foreground threads have finished. They don't get a chance to complete.

```csharp
// This foreground thread keeps the process alive for 10 seconds
// even after Main() returns
Thread fg = new Thread(() =>
{
    Thread.Sleep(10_000);
    Console.WriteLine("Foreground done"); // WILL print
});
fg.IsBackground = false; // default — foreground
fg.Start();

// This background thread is killed when all foreground threads finish
Thread bg = new Thread(() =>
{
    Thread.Sleep(10_000);
    Console.WriteLine("Background done"); // may NEVER print
});
bg.IsBackground = true;
bg.Start();
```

### Defaults

| Thread source                  | Default                          |
| ------------------------------ | -------------------------------- |
| `new Thread(...)`              | **Foreground**                   | 
| `ThreadPool.QueueUserWorkItem` | Background                       |
| `Task.Run`                     | Background (runs on thread pool) |

The primary thread is always a foreground thread. Worker threads created with `new Thread(...)` are also foreground by default — you must explicitly set `IsBackground = true` to make them background.

### Two Different Classifications — Don't Confuse Them

These are **two independent axes**. They answer different questions:

| Classification | Question it answers | Values |
|---|---|---|
| Primary vs Worker | **Who created it?** | Primary = created by the OS when the process started. Worker = created by your code |
| Foreground vs Background | **Does it keep the process alive?** | Foreground = yes, process waits. Background = no, killed on exit |

They can combine in any way:

| Thread | Primary/Worker | Foreground/Background |
|---|---|---|
| The main thread running `Main()` | Primary | Foreground |
| `new Thread(DoWork)` (default) | Worker | Foreground |
| `new Thread(DoWork) { IsBackground = true }` | Worker | Background |
| Thread pool thread (`Task.Run`) | Worker | Background |

```ad-warning
A common mistake: assuming "worker thread" means "background thread." By default, `new Thread(...)` creates a **foreground** worker thread — it will keep your process alive even if `Main()` finishes. If you want fire-and-forget behavior, you must set `IsBackground = true`.
```

### When to Use Each

| Scenario | Use |
|---|---|
| Work that **must complete** before the app exits (saving data, flushing logs) | Foreground |
| Work that is **okay to abandon** if the app shuts down (monitoring, periodic cleanup) | Background |
| Long-running listener or polling loop | Background — otherwise the process can never exit |


---

## What Threads Share (and What They Don't)

| Shared across all threads     | Private to each thread                                    |
| ----------------------------- | --------------------------------------------------------- |
| Heap memory (objects, arrays) | Stack (local variables, method calls)                     |
| Static / global variables     | Program counter (where it is in the code)                 | 
| Open file handles, sockets    | Thread-local storage (`[ThreadStatic]`, `ThreadLocal<T>`) |
| Loaded assemblies / DLLs      | Exception context                                         |

Because threads share the heap and static data, **concurrent access** to shared state can cause race conditions, deadlocks, and data corruption — this is why multithreaded programming requires synchronization.


---

## Why Use Multiple Threads?

A single-threaded application has a fundamental limitation: **it can only do one thing at a time.** If the primary thread is busy (e.g., computing, waiting on I/O), the entire application is blocked.

| Problem | Single-threaded | Multi-threaded |
|---|---|---|
| Long computation | UI freezes until done | Worker thread computes, UI stays responsive |
| Downloading a file | App hangs until download finishes | Background thread downloads, app continues |
| Processing multiple requests | One at a time, sequentially | Multiple threads handle requests concurrently |

```csharp
// Single-threaded — UI freezes during the computation
void Button_Click(object sender, EventArgs e)
{
    var result = ExpensiveCalculation(); // blocks the UI thread
    label.Text = result.ToString();
}

// Multi-threaded — UI stays responsive
async void Button_Click(object sender, EventArgs e)
{
    var result = await Task.Run(() => ExpensiveCalculation()); // runs on a worker thread
    label.Text = result.ToString(); // back on the UI thread
}
```


---

## The Cost of Threads

Threads are not free. Each thread consumes:

- **~1 MB of stack memory** (default on Windows)
- **OS kernel resources** for scheduling
- **CPU time** for context switching between threads

Creating hundreds or thousands of threads is wasteful. This is why .NET provides the **ThreadPool** and **async/await** — they reuse a small pool of threads efficiently rather than creating new ones for every piece of work.


---

## How Threads Are Created in .NET

There are several ways to work with threads, from low-level to high-level:

| Approach                       | Level  | When to use                                       |
| ------------------------------ | ------ | ------------------------------------------------- |
| `Thread` class                 | Low    | Full control over thread lifetime, name, priority |
| `ThreadPool.QueueUserWorkItem` | Medium | Short fire-and-forget background work             |
| `Task.Run`                     | High   | Modern async work (preferred for most scenarios)  |
| `async / await`                | High   | I/O-bound or CPU-bound work without blocking      |

```csharp
// Low-level: manual Thread
Thread t = new Thread(() => Console.WriteLine("Worker thread"));
t.Start();

// Medium: ThreadPool
ThreadPool.QueueUserWorkItem(_ => Console.WriteLine("Pool thread"));

// High-level: Task (preferred)
await Task.Run(() => Console.WriteLine("Task thread"));
```

```ad-note
The `ProcessThread` type in `System.Diagnostics` is **not** used to create or manage threads. It is a diagnostic tool for inspecting the active OS threads within a running process. See [[ProcessThread]].
```
