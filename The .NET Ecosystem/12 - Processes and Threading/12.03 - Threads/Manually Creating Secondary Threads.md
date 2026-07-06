---
tags:
 - csharp
 - threading
---

## The Process

Creating a thread manually follows five steps:

1. **Create a method** to be the entry point for the new thread
2. **Create a delegate** — `ThreadStart` (no params) or `ParameterizedThreadStart` (one `object?` param) — pointing to that method
3. **Create a `Thread` object**, passing the delegate to its constructor
4. **Configure the thread** — name, priority, background/foreground
5. **Call `Thread.Start()`** — the thread begins executing the entry point method as soon as possible

```
Main Thread                             Worker Thread
    │                                        
    │  1. Define DoWork()                    
    │  2. new ThreadStart(DoWork)            
    │  3. new Thread(delegate)               
    │  4. Set Name, Priority, etc.           
    │  5. thread.Start()                     
    │──────────────────────────────────►  DoWork() begins
    │  (main continues immediately)          │
    │                                        │ executing...
    │                                        │
    │  thread.Join()                         │
    │  ◄─── blocks until ─────────────────  DoWork() returns
    │                                     thread is now dead
    │  continues...
```


---

## Step by Step — `ThreadStart` (No Parameters)

```csharp
// Step 1: define the entry point method
void DoWork()
{
    Console.WriteLine($"Running on thread {Thread.CurrentThread.ManagedThreadId}");
    for (int i = 0; i < 5; i++)
    {
        Console.WriteLine($"  Working... {i}");
        Thread.Sleep(500);
    }
    Console.WriteLine("Work complete.");
}

// Step 2: create the delegate
ThreadStart workDelegate = new ThreadStart(DoWork);

// Step 3: create the Thread object
Thread workerThread = new Thread(workDelegate);

// Step 4: configure
workerThread.Name = "BackgroundWorker";
workerThread.IsBackground = true;
workerThread.Priority = ThreadPriority.Normal;

// Step 5: start
workerThread.Start();

// Optionally wait for completion
workerThread.Join();
```

### The `ThreadStart` Delegate

`ThreadStart` is defined as:

```csharp
public delegate void ThreadStart();
```

It represents a method with **no parameters and no return value**. The method just runs — if you need to get data back, you must write the result to a shared field or use a callback.

### Shorthand — Compiler Infers the Delegate

In practice, steps 2-3 are shortened because the compiler infers the delegate type:

```csharp
// These are all equivalent:
Thread t = new Thread(new ThreadStart(DoWork));  // explicit delegate
Thread t = new Thread(DoWork);                    // compiler infers ThreadStart
Thread t = new Thread(() =>                       // lambda — no separate method needed
{
    Console.WriteLine("Working on a new thread");
});
```


---

## Step by Step — `ParameterizedThreadStart` (With a Parameter)

When the entry point needs input, use `ParameterizedThreadStart`:

```csharp
public delegate void ParameterizedThreadStart(object? obj);
```

It accepts a single `object?` parameter — you must cast it inside the method:

```csharp
// Step 1: entry point with parameter
void ProcessFile(object? data)
{
    string path = (string)data!;
    Console.WriteLine($"Processing {path} on thread {Thread.CurrentThread.ManagedThreadId}");
    Thread.Sleep(2000); // simulate work
    Console.WriteLine($"Finished processing {path}");
}

// Steps 2-3
Thread t = new Thread(new ParameterizedThreadStart(ProcessFile));

// Step 4
t.Name = "FileProcessor";

// Step 5: pass the argument to Start()
t.Start(@"C:\Data\input.csv");
```

```ad-warning
The parameter is always `object?` — you must cast it inside the method. This is a pre-generics design from .NET 1.0. For type safety, prefer a lambda closure:

```csharp
string path = @"C:\Data\input.csv";
int maxRetries = 3;
// Lambda captures both variables with their real types — no casting
Thread t = new Thread(() => ProcessFile(path, maxRetries));
t.Start();
```

This also solves the limitation of `ParameterizedThreadStart` only accepting **one** parameter — a lambda can capture as many variables as needed.


---

## `ThreadStart` vs `ParameterizedThreadStart`

| Feature | `ThreadStart` | `ParameterizedThreadStart` |
|---|---|---|
| Signature | `void Method()` | `void Method(object? obj)` |
| How to pass data | Through closures or shared fields | Via `Thread.Start(arg)` |
| Type safety | N/A (no parameter) | Weak — must cast from `object?` |
| Multiple parameters | N/A | No — only one `object?` (workaround: use a tuple or custom object) |
| Modern alternative | Lambda with no captures | Lambda with captures (preferred) |


---

## Thread Lifecycle — What Happens When

A `Thread` object goes through several states from creation to death:

```
                    ┌──────────┐
       new Thread() │ Unstarted │
                    └─────┬────┘
                          │ .Start()
                    ┌─────▼────┐
                    │ Running   │◄──────────────────────┐
                    └─────┬────┘                        │
                          │                             │
              ┌───────────┼───────────┐                 │
              │           │           │                 │
       Thread.Sleep()  lock/Wait   I/O blocked          │
              │           │           │                 │
        ┌─────▼───────────▼───────────▼────┐            │
        │         WaitSleepJoin            │            │
        └─────────────┬───────────────────┘            │
                      │ sleep expires / lock acquired   │
                      └────────────────────────────────┘
                          │
                    entry point method returns
                          │
                    ┌─────▼────┐
                    │ Stopped   │  ← thread is dead, cannot be restarted
                    └──────────┘
```

You can check the state at any time:

```csharp
Thread t = new Thread(DoWork);
Console.WriteLine(t.ThreadState); // Unstarted

t.Start();
Console.WriteLine(t.ThreadState); // Running (or WaitSleepJoin if it called Sleep)

t.Join();
Console.WriteLine(t.ThreadState); // Stopped
```

```ad-danger
A stopped thread **cannot be restarted**. Calling `Start()` on a thread that has already run and finished throws `ThreadStateException`. You must create a new `Thread` object.
```


---

## Exception Handling in Threads

An unhandled exception in a secondary thread **terminates the entire process** (since .NET 2.0). The exception does not propagate to the calling thread — it crashes everything:

```csharp
Thread t = new Thread(() =>
{
    throw new InvalidOperationException("Something broke!");
    // This kills the ENTIRE process — not just this thread
});
t.Start();
```

You must handle exceptions **inside** the thread's entry point:

```csharp
Thread t = new Thread(() =>
{
    try
    {
        DoRiskyWork();
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Worker thread error: {ex.Message}");
        // Log it, set a flag, signal the main thread — but don't let it escape
    }
});
t.Start();
```

```ad-note
This is one advantage of `Task.Run` over raw `Thread` — tasks capture exceptions and re-throw them when you `await` or call `.Result`, so the calling code can handle them naturally. With raw threads, you must do all exception handling yourself inside the entry point.
```


---

## Getting Results Back from a Thread

`ThreadStart` and `ParameterizedThreadStart` both return `void` — there's no built-in way to return a value. Common patterns:

### Shared Field

```csharp
string? result = null;

Thread t = new Thread(() =>
{
    result = ComputeExpensiveThing(); // write to shared field
});
t.Start();
t.Join(); // wait for thread to finish writing

Console.WriteLine(result); // safe to read after Join
```

### Callback Delegate

```csharp
void ComputeOnThread(Action<int> callback)
{
    Thread t = new Thread(() =>
    {
        int answer = ExpensiveCalculation();
        callback(answer); // call back with the result
    });
    t.Start();
}

ComputeOnThread(result =>
{
    Console.WriteLine($"Got result: {result}");
});
```

```ad-note
Both patterns are clunky — this is exactly the problem that `Task<T>` was designed to solve:

```csharp
// Modern equivalent — cleaner, safer, composable
int result = await Task.Run(() => ExpensiveCalculation());
```


---

## Complete Example — Multiple Workers

```csharp
void PrintNumbers(object? data)
{
    string name = (string)data!;
    for (int i = 0; i < 5; i++)
    {
        Console.WriteLine($"  {name}: {i}");
        Thread.Sleep(300);
    }
}

Console.WriteLine($"Main thread: {Thread.CurrentThread.ManagedThreadId}");

Thread t1 = new Thread(PrintNumbers) { Name = "Worker-A" };
Thread t2 = new Thread(PrintNumbers) { Name = "Worker-B" };

t1.Start("Alpha");
t2.Start("Beta");

// Wait for both to finish before continuing
t1.Join();
t2.Join();

Console.WriteLine("Both workers finished.");
```

Output (interleaved — order varies each run):

```
Main thread: 1
  Alpha: 0
  Beta: 0
  Alpha: 1
  Beta: 1
  Alpha: 2
  Beta: 2
  ...
Both workers finished.
```

```ad-note
Without `Join()`, the main thread would continue immediately — potentially exiting the program before the workers finish (if they're background threads). See [[The System.Threading.Thread class]] for foreground vs background behavior.
```


---

## Summary — When to Use Manual Threads vs Modern Alternatives

| Approach | Use when |
|---|---|
| `new Thread(...)` | You need a dedicated, named, long-running thread with specific priority or foreground/background control |
| `ThreadPool.QueueUserWorkItem` | Short fire-and-forget work, don't need a result |
| `Task.Run` | Short background work where you want a result, exception propagation, or `await` support |
| `async/await` | I/O-bound work (no thread consumed) or CPU-bound work via `Task.Run` |

Manual thread creation is the lowest-level option. It gives the most control but requires you to handle everything yourself — parameters, results, exceptions, synchronization. Modern .NET code almost always uses `Task` instead.
