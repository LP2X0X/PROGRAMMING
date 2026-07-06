

---
tags:
 - csharp
 - concurrency
 - threading
---

## How Everything Connects

The concurrency landscape in .NET has multiple layers. Each solves a different problem, and they compose — not replace — each other.

```
┌─────────────────────────────────────────────────────────┐
│                      TASK                               │
│         (just a promise — "something will complete")    │
│                                                         │
│   Properties: IsCompleted, Result, IsFaulted            │
│   Can be awaited with: await                            │
│                                                         │
│   A Task does NOT do work. It TRACKS work.              │
└─────────────┬──────────────────────┬────────────────────┘
              │                      │
     Who fulfills the promise?       │
              │                      │
    ┌─────────▼──────────┐  ┌───────▼────────────────────┐
    │   CPU-bound work   │  │      I/O-bound work        │
    │                    │  │                             │
    │   Task.Run(...)    │  │  httpClient.GetStringAsync  │
    │                    │  │  stream.ReadAsync            │
    │   Queues work onto │  │  db.ExecuteAsync            │
    │   the ThreadPool   │  │                             │
    │         │          │  │  No threads involved.       │
    │         ▼          │  │  OS + hardware do the work. │
    │   ┌────────────┐   │  │  OS signals .NET when done. │
    │   │ ThreadPool │   │  │                             │
    │   │            │   │  │  Task completes via         │
    │   │ Manages a  │   │  │  callback, not a thread.   │
    │   │ pool of    │   │  │                             │
    │   │ reusable   │   │  └─────────────────────────────┘
    │   │ Threads    │   │
    │   │     │      │   │
    │   │     ▼      │   │
    │   │ ┌────────┐ │   │
    │   │ │ Thread │ │   │
    │   │ │ Thread │ │   │
    │   │ │ Thread │ │   │
    │   │ └────────┘ │   │
    │   └────────────┘   │
    └────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                    async / await                         │
│                                                         │
│   async  = tells compiler to build a state machine      │
│   await  = suspend the METHOD, release the THREAD       │
│                                                         │
│   Does NOT create threads.                              │
│   Does NOT do work.                                     │
│   Just controls HOW YOU WAIT for a Task to complete.    │
└─────────────────────────────────────────────────────────┘
```


---

## Concept Summary

| Thing        | What it is                                                             | Creates threads?         |
| ------------ | ---------------------------------------------------------------------- | ------------------------ |
| `Thread`     | An actual OS thread that runs code                                     | It **is** a thread       |
| `ThreadPool` | .NET manages a pool of reusable Threads for you                        | Manages existing threads |
| `Task.Run()` | Puts CPU work onto the ThreadPool, gives you a Task to track it        | Uses ThreadPool threads  |
| `Task`       | A promise object — tracks completion, holds result. Thread-agnostic.   | **No**                   |
| `async`      | Compiler hint: "build a state machine for this method"                 | **No**                   |
| `await`      | "Suspend this method, free the thread, resume when the Task completes" | **No**                   |


---

## The Abstraction Layers

```
Layer 0:  Thread          — you create it, manage it, destroy it
Layer 1:  ThreadPool      — reusable pool of threads managed by .NET
Layer 2:  Task.Run()      — convenient way to queue work onto the ThreadPool
Layer 3:  async/await     — controls how you WAIT, not how work runs
```

- **Layer 0 ([[The System.Threading.Thread class|Thread]])** — raw OS threads. Full control, full responsibility. You create it, start it, join it, handle exceptions manually.
- **Layer 1 ([[The CLR Thread Pool|ThreadPool]])** — .NET recycles a pool of threads so you don't pay creation cost every time. You queue work; it picks a thread.
- **Layer 2 ([[The Task Class|Task.Run]])** — wraps ThreadPool with a `Task` handle so you can track completion, get results, and chain operations.
- **Layer 3 ([[A First Look at the C# async and await Keywords|async/await]])** — a language-level pattern for writing non-blocking code that reads like synchronous code. Works with both CPU-bound (via Task.Run) and I/O-bound (no threads) operations.


---

## Common Misconceptions

| Misconception                       | Reality                                                                |
| ----------------------------------- | ---------------------------------------------------------------------- |
| "`async` creates a new thread"      | `async` creates zero threads — it builds a state machine               |
| "`await` pauses the thread"         | `await` releases the thread and pauses the **method**                  |
| "`Task` always uses a thread"       | `Task` is just a promise — I/O tasks use no threads at all             | 
| "`Task.Run` is the same as `async`" | `Task.Run` queues CPU work on ThreadPool; `async` is a waiting pattern |


---

## Section Map

| Section                           | Covers                                                                            |
| --------------------------------- | --------------------------------------------------------------------------------- |
| [[12.01 - Processes]]             | OS processes, `Process` class, `ProcessStartInfo`                                 | 
| [[12.02 - AppDomains]]            | `AssemblyLoadContext`, isolation boundaries                                       |
| [[12.03 - Threads]]               | `Thread`, `ThreadPool`, synchronization (`lock`, `Monitor`, `Mutex`, `Semaphore`) |
| [[12.04 - Task Parallel Library]] | `Task`, `Parallel`, PLINQ — CPU-bound parallelism                                 |
| [[12.05 - Async Programming]]     | `async`/`await`, state machines, `SynchronizationContext` — I/O-bound concurrency |
