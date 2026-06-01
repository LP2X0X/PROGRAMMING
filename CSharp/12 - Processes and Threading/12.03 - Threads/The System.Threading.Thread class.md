---
tags:
 - csharp
 - threading
---


`Thread` is the most primitive type in `System.Threading` — an object-oriented wrapper around an OS thread. It gives you full control over creating, starting, naming, prioritizing, and managing individual threads within a process.

```ad-note
For most modern .NET work, you should use `Task.Run` or `async/await` instead of raw `Thread`. The `Thread` class is for when you need direct control — long-running background threads, setting thread names for debugging, or controlling thread priority.
```


---

## Creating and Starting a Thread

### Parameterless — `ThreadStart`

```csharp
void PrintNumbers()
{
    for (int i = 0; i < 5; i++)
    {
        Console.WriteLine($"Thread {Thread.CurrentThread.ManagedThreadId}: {i}");
        Thread.Sleep(100);
    }
}

Thread t = new Thread(new ThreadStart(PrintNumbers));
// Or simply (compiler infers the delegate):
Thread t = new Thread(PrintNumbers);

t.Start();
t.Join(); // block until thread t finishes
```

### With a Parameter — `ParameterizedThreadStart`

```csharp
void PrintMessage(object? data)
{
    string msg = (string)data!;
    Console.WriteLine($"Thread says: {msg}");
}

Thread t = new Thread(new ParameterizedThreadStart(PrintMessage));
t.Start("Hello from another thread");
```

```ad-warning
`ParameterizedThreadStart` takes a single `object?` parameter — you must cast it. This is a pre-generics design. For type safety, prefer a lambda that captures the variable:
```

```csharp
string message = "Hello";
Thread t = new Thread(() => PrintMessage(message)); // type-safe via closure
t.Start();
```


---

## Key Static Members

Static members of `Thread` operate on the **currently executing** thread:

| Member                        | Purpose                                                                                                 |
| ----------------------------- | ------------------------------------------------------------------------------------------------------- |
| `Thread.CurrentThread`        | Returns the `Thread` object for the thread running this code                                            |
| `Thread.Sleep(ms)`            | Pauses the current thread for the specified duration                                                    | 
| `Thread.Sleep(0)`             | Yields the current time slice — lets other threads run, then immediately re-enters the scheduling queue |
| `Thread.Yield()`              | Yields to another thread on the same CPU core (more targeted than `Sleep(0)`)                           |
| `Thread.SpinWait(iterations)` | Busy-waits for a very short time without yielding (used in low-level synchronization)                   |
| `Thread.MemoryBarrier()`      | Prevents CPU/compiler from reordering reads and writes across this point                                |

```csharp
Console.WriteLine($"Current thread ID: {Thread.CurrentThread.ManagedThreadId}");

Thread.Sleep(1000); // pause for 1 second
```


---

## Key Instance Properties

| Property | Type | Description |
|---|---|---|
| `Name` | `string?` | A human-readable name — shows up in the debugger. Can only be set **once** |
| `ManagedThreadId` | `int` | The CLR's ID for this thread (not the same as the OS thread ID) |
| `IsAlive` | `bool` | `true` if the thread has been started and hasn't terminated |
| `IsBackground` | `bool` | Background threads don't keep the process alive (see below) |
| `Priority` | `ThreadPriority` | Hint to the OS scheduler about relative importance |
| `ThreadState` | `ThreadState` | Current state: `Running`, `WaitSleepJoin`, `Stopped`, etc. |

```csharp
Thread t = new Thread(DoWork);
t.Name = "DataProcessor";
t.IsBackground = true;
t.Priority = ThreadPriority.BelowNormal;
t.Start();

Console.WriteLine($"Thread '{t.Name}' is alive: {t.IsAlive}");
```


---

## Foreground vs Background Threads

This is one of the most important distinctions:

- **Foreground threads** keep the process alive. The process will not exit until all foreground threads have finished.
- **Background threads** are killed automatically when all foreground threads have finished.

```csharp
Thread t = new Thread(() =>
{
    Thread.Sleep(10_000);
    Console.WriteLine("This may never print");
});

t.IsBackground = true;  // process won't wait for this thread
t.Start();

// If Main() returns immediately, the process exits
// and the background thread is killed — it never prints
```

By default, threads created with `new Thread(...)` are **foreground**. Thread pool threads (used by `Task.Run`) are always **background**.

| Source | Default |
|---|---|
| `new Thread(...)` | Foreground |
| `ThreadPool` / `Task.Run` | Background |


---

## Thread Priority

A hint to the OS scheduler about how much CPU time this thread should get relative to other threads:

| Priority | Value |
|---|---|
| `ThreadPriority.Highest` | 4 |
| `ThreadPriority.AboveNormal` | 3 |
| `ThreadPriority.Normal` | 2 (default) |
| `ThreadPriority.BelowNormal` | 1 |
| `ThreadPriority.Lowest` | 0 |

```csharp
Thread worker = new Thread(CrunchNumbers);
worker.Priority = ThreadPriority.BelowNormal; // don't starve other threads
worker.Start();
```

```ad-warning
Thread priority is a **hint**, not a guarantee. The OS can and will override it based on system conditions. High-priority threads can starve lower-priority ones. Avoid `Highest` unless you have a specific, measured need.
```


---

## Joining a Thread — Waiting for Completion

`Join()` blocks the calling thread until the target thread finishes:

```csharp
Thread t = new Thread(() =>
{
    Thread.Sleep(2000);
    Console.WriteLine("Worker done");
});
t.Start();

Console.WriteLine("Waiting for worker...");
t.Join();          // blocks here until t finishes
Console.WriteLine("Worker has finished");

// With a timeout:
bool finished = t.Join(5000);  // wait up to 5 seconds
if (!finished)
    Console.WriteLine("Thread is still running");
```


---

## Key Instance Methods

| Method | Description |
|---|---|
| `Start()` | Begins execution of the thread |
| `Start(object?)` | Begins execution, passing a parameter (for `ParameterizedThreadStart`) |
| `Join()` | Blocks the calling thread until this thread terminates |
| `Join(int ms)` | Blocks up to `ms` milliseconds, returns `true` if the thread finished |
| `Interrupt()` | Throws `ThreadInterruptedException` in the target thread if it's in a `WaitSleepJoin` state |
| `Abort()` | **Removed in .NET 5+** — throws `PlatformNotSupportedException`. Was dangerous even in .NET Framework |

```ad-danger
`Thread.Abort()` was removed in .NET Core / .NET 5+ because it could corrupt application state by throwing an exception at an arbitrary point in code. Use **cooperative cancellation** (`CancellationToken`) instead.
```


---

## Thread vs Task.Run — When to Use Each

| Use `Thread` when | Use `Task.Run` when |
|---|---|
| You need a dedicated, long-running thread | Short-lived background work |
| You need to set the thread name for debugging | You want to return a result (`Task<T>`) |
| You need to control foreground/background | You want to `await` the completion |
| You need fine-grained priority control | You want automatic thread pool management |

```csharp
// Thread — long-running dedicated worker
Thread listener = new Thread(ListenForConnections);
listener.Name = "ConnectionListener";
listener.IsBackground = true;
listener.Start();

// Task.Run — short work on the thread pool
int result = await Task.Run(() => ComputeExpensiveThing());
```
