---
tags:
 - csharp
 - async
 - tpl
---

## The Problem async/await Solves

Most of the work applications do involves **waiting** — waiting for a database query, an HTTP response, a file read, a network socket. In a synchronous world, the thread that starts the I/O operation **blocks** until it completes. That thread is alive, consuming ~1 MB of stack memory and an OS kernel object, but doing absolutely nothing.

With async/await, the thread is **freed** while the I/O is in progress. The method **suspends** at the `await` point and **resumes** when the result arrives — potentially on a different thread. No thread is wasted waiting.

```csharp
// Synchronous — thread blocked for ~500ms doing nothing:
string html = httpClient.GetString("https://example.com");

// Asynchronous — thread freed during the ~500ms wait:
string html = await httpClient.GetStringAsync("https://example.com");
```

This is especially critical in two scenarios:
- **UI applications** — blocking the UI thread freezes the entire application
- **Server applications** — each blocked thread under load means fewer threads available to serve other requests, leading to [[The CLR Thread Pool|thread pool starvation]]


---

## The Core Idea

`async` and `await` are **compiler features**, not runtime magic. The `async` keyword tells the compiler to transform the method into a state machine. The `await` keyword marks a suspension point — where the method can pause and later resume.

```csharp
async Task<string> FetchAndProcessAsync(string url)
{
    var raw = await _httpClient.GetStringAsync(url);  // Suspend here if not done
    var processed = Transform(raw);                   // Resumes here when data arrives
    await _repository.SaveAsync(processed);           // Suspend again
    return processed;
}
```

The method reads top-to-bottom like synchronous code, but underneath, the compiler breaks it into segments separated by each `await`. Each segment runs when the previous awaited operation completes. See [[The Task Class]] for how `Task` represents these operations.


---

## What async Actually Does — The State Machine

The `async` keyword does **not** make a method "run in the background." It instructs the compiler to rewrite the method into a **state machine struct** that implements `IAsyncStateMachine`. Each `await` expression becomes a state transition. Local variables are hoisted to fields on the struct so they survive across suspensions.

```
Your code:                         Compiler generates:
                                   
async Task<T> Method()             struct <Method>d__0 : IAsyncStateMachine
{                                  {
    var a = await X();                 int state;     // which await are we at?
    var b = await Y(a);                T_a a_field;   // hoisted local
    return b;                          T_b b_field;   // hoisted local
}                                      
                                       void MoveNext() { switch(state) ... }
                                   }
```

This is why `ref` locals and `Span<T>` can't live across an `await` — they can't be struct fields.

Deep dive: [[The Async State Machine]]


---

## What await Does — Suspension and Resumption

`await` checks if the task is already complete. If yes, execution continues synchronously (no suspension, no thread switch — the fast path). If not, it:

1. Captures the current [[SynchronizationContext and ConfigureAwait|SynchronizationContext]]
2. Registers a continuation (the rest of the method) on the task
3. **Returns from the method** — the calling thread is freed

When the task completes later, the continuation is posted back to the captured context (e.g., the UI thread in WPF) or to a thread pool thread.

Deep dive: [[SynchronizationContext and ConfigureAwait]]


---

## Async Return Types

The return type of an async method determines how the caller observes completion, how exceptions propagate, and what the performance characteristics are:

| Return type | When to use | Key characteristic |
|---|---|---|
| `Task<T>` | Method produces a value | Default choice — safe, awaitable |
| `Task` | Method has no return value | Async equivalent of `void` |
| `ValueTask<T>` | Hot path, often sync completion | Avoids heap allocation on fast path |
| `async void` | Event handlers **only** | Cannot be awaited — exceptions crash the process |

The **naming convention** is to suffix async methods with `Async`: `GetUserAsync`, `SaveOrderAsync`, `ReadFileAsync`. This signals to callers that the method returns a `Task` that should be `await`ed.

Deep dive: [[Async Return Types]]


---

## Exception Handling

`await` re-throws exceptions from the faulted task, so `try`/`catch` works naturally — just like synchronous code:

```csharp
try
{
    var data = await GetDataAsync();
}
catch (HttpRequestException ex)
{
    // Works exactly as you'd expect
}
```

The key nuances: `await` unwraps exceptions directly, while `.Result`/`.Wait()` wrap them in `AggregateException`. And `async void` methods don't capture exceptions on a task at all — they post to the `SynchronizationContext` and usually crash the process.

Deep dive: [[Exception Handling in Async Code]]


---

## Common Pitfalls

The most frequent async bugs, in order of how often they cause pain:

1. **Calling `.Result` or `.Wait()`** — deadlocks in UI apps, thread pool starvation in server apps
2. **Forgetting to `await`** — silent fire-and-forget with lost exceptions (compiler warns with CS4014)
3. **Not going "async all the way"** — one sync method in the call chain forces sync-over-async somewhere
4. **Eliding `async`/`await` with `using`** — resource disposed before the async operation completes
5. **Using `async void` outside event handlers** — unobservable exceptions, untestable methods

Deep dive: [[Common Async Pitfalls]]


---

## Quick Reference Table

| Concept | Key point | Detail note |
|---|---|---|
| `async` keyword | Compiler directive — transforms method into state machine | [[The Async State Machine]] |
| `await` keyword | Suspension point — captures context, frees thread, resumes later | [[SynchronizationContext and ConfigureAwait]] |
| SynchronizationContext | Controls which thread resumes after `await` | [[SynchronizationContext and ConfigureAwait]] |
| ConfigureAwait(false) | Opts out of context capture — use in library code | [[SynchronizationContext and ConfigureAwait]] |
| Task / Task\<T> | Default return types for async methods | [[Async Return Types]] |
| ValueTask\<T> | Avoids allocation when sync completion is common | [[Async Return Types]] |
| async void | Only for event handlers — exceptions crash the process | [[Async Return Types]] |
| try/catch with await | Works naturally — await unwraps exceptions | [[Exception Handling in Async Code]] |
| .Result / .Wait() | Blocks thread, risks deadlock, wraps in AggregateException | [[Common Async Pitfalls]] |
| Async suffix naming | Convention: `MethodAsync()` for Task-returning methods | [[Async Return Types]] |


---

## See Also

- [[The Task Class]] — the Task abstraction that async/await is built on
- [[The CLR Thread Pool]] — where async continuations execute
- [[Data Parallelism with the Parallel Class]] — CPU-bound parallelism (distinct from async I/O)
