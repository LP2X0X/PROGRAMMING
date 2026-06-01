---
tags:
 - csharp
 - threading
---

`System.Threading` is the namespace that contains all the low-level and mid-level types for working with threads, synchronization, and concurrent execution in .NET.

```csharp
using System.Threading;
```


---

## Key Types at a Glance

### Thread Management

| Type                       | Purpose                                                                                                                                                              |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Thread`                   | The most primitive type — an OOP wrapper around an OS thread. Create, start, suspend, name, and manage individual threads. See [[The System.Threading.Thread class]] |
| `ThreadPool`               | A managed pool of reusable worker threads. Avoids the cost of creating/destroying threads for short work. See [[The CLR Thread Pool]]                                |
| `ThreadStart`              | Delegate for parameterless thread entry points (`void Method()`)                                                                                                     | 
| `ParameterizedThreadStart` | Delegate for thread entry points that accept one `object` parameter                                                                                                  |

### Synchronization Primitives

| Type                                        | Purpose                                                                                 |
| ------------------------------------------- | --------------------------------------------------------------------------------------- |
| `Monitor`                                   | The mechanism behind the `lock` keyword — mutual exclusion with `Enter`/`Exit`. See [[Lock and Monitor]]          |
| `Mutex`                                     | Like `Monitor` but works **across processes** (named mutex = system-wide lock). See [[Mutex]]          |
| `Semaphore` / `SemaphoreSlim`               | Allows up to N threads to access a resource concurrently. See [[Semaphore and SemaphoreSlim]]                                | 
| `ManualResetEvent` / `ManualResetEventSlim` | A signal — threads wait until another thread "sets" it. Stays open until manually reset. See [[Signaling Primitives]] |
| `AutoResetEvent`                            | Like `ManualResetEvent` but automatically resets after releasing one waiting thread. See [[Signaling Primitives]]     |
| `ReaderWriterLockSlim`                      | Allows many readers OR one writer — efficient for read-heavy shared data. See [[ReaderWriterLockSlim]]                |
| `SpinLock`                                  | Busy-wait lock for extremely short critical sections (avoids context switch overhead). See [[SpinLock Barrier and CountdownEvent]]   |
| `Barrier`                                   | Coordinates multiple threads to all reach a point before any proceed. See [[SpinLock Barrier and CountdownEvent]]                    |
| `CountdownEvent`                            | A signal that fires after being signaled N times. See [[SpinLock Barrier and CountdownEvent]]                                        |

### Atomic Operations and Memory

| Type | Purpose |
|---|---|
| `Interlocked` | Atomic read-modify-write operations (`Increment`, `Exchange`, `CompareExchange`). See [[Volatile and Atomic Operations]] |
| `Volatile` | Static methods for volatile reads/writes (alternative to the `volatile` keyword) |

### Thread-Local and Async-Local Storage

| Type | Purpose |
|---|---|
| `ThreadLocal<T>` | A value that is private to each thread — each thread gets its own independent copy |
| `AsyncLocal<T>` | A value that flows with `ExecutionContext` across `await` boundaries (the modern replacement for `ThreadLocal` in async code) |

### Other Useful Types

| Type | Purpose |
|---|---|
| `Timer` | Executes a callback at specified intervals on a thread pool thread |
| `CancellationTokenSource` / `CancellationToken` | Cooperative cancellation — signal a task to stop gracefully |
| `ExecutionContext` | The ambient data (security, culture, AsyncLocal values) that flows with async execution |
| `SynchronizationContext` | Controls which thread a continuation runs on (used by UI frameworks to marshal back to the UI thread) |


---

## How the Types Relate

```
                    ┌───────────────────┐
                    │  async / await    │  ← what you normally use
                    └────────┬──────────┘
                             │ uses
                    ┌────────▼──────────┐
                    │    Task / Task<T> │  ← System.Threading.Tasks
                    └────────┬──────────┘
                             │ schedules on
                    ┌────────▼──────────┐
                    │    ThreadPool     │  ← reuses a few threads
                    └────────┬──────────┘
                             │ manages
                    ┌────────▼──────────┐
                    │     Thread        │  ← OS thread wrapper
                    └───────────────────┘
```

Most application code lives at the top (`async/await` and `Task`). The lower layers exist for when you need direct control — long-running threads, precise synchronization, or performance-critical lock-free code.

```ad-note
`System.Threading.Tasks` is a separate namespace from `System.Threading` but builds directly on top of it. `Task.Run` schedules work on the `ThreadPool`, which manages `Thread` objects internally.
```
