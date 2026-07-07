---
tags:
 - csharp
 - async
 - tpl
 - synchronization
---

## What Is SynchronizationContext?

**`SynchronizationContext`** is an abstraction that represents a "place" where code should run. It answers one question: **after an `await` completes, which thread (or thread group) should resume executing the rest of the method?**

Every `await` captures the current `SynchronizationContext` (or `TaskScheduler`, if no context is present) **before** suspending. When the awaited task completes, the continuation (the code after `await`) is **posted back** to that captured context.

```csharp
// In a WPF application:
async void Button_Click(object sender, RoutedEventArgs e)
{
    // Running on the UI thread — SynchronizationContext is the
    // DispatcherSynchronizationContext
    
    StatusLabel.Text = "Downloading...";  // ✓ UI thread
    
    var data = await DownloadAsync(url);
    // ↑ Captures DispatcherSynchronizationContext before suspending
    
    // After await: posted BACK to the UI thread via the captured context
    StatusLabel.Text = $"Got {data.Length} bytes";  // ✓ Still UI thread
}
```

Without this mechanism, the continuation would run on whatever thread pool thread happened to complete the I/O — and touching UI controls from a non-UI thread would throw an `InvalidOperationException`.


---

## SynchronizationContext by Application Type

Different application types install different synchronization contexts:

| Application Type | SynchronizationContext | `Post()` behavior |
|---|---|---|
| **WinForms** | `WindowsFormsSynchronizationContext` | Marshals to the UI message loop via `Control.BeginInvoke` |
| **WPF** | `DispatcherSynchronizationContext` | Marshals to the `Dispatcher` thread |
| **ASP.NET (classic)** | `AspNetSynchronizationContext` | Restores `HttpContext` on a pool thread (not a single thread) |
| **ASP.NET Core** | **None** (`null`) | No context — resumes on any thread pool thread |
| **Console app** | **None** (`null`) | No context — resumes on any thread pool thread |
| **xUnit / NUnit** | Custom or None | Test frameworks may install their own |

```ad-note
ASP.NET Core **deliberately removed** the synchronization context that classic ASP.NET had. The old `AspNetSynchronizationContext` was a source of deadlocks and unnecessary serialization. Without it, continuations resume on any available pool thread — which is exactly what you want for high-throughput server code.
```

You can inspect the current context at any time:

```csharp
var ctx = SynchronizationContext.Current;
Console.WriteLine(ctx?.GetType().Name ?? "null (no context)");
```


---

## How `await` Captures and Uses the Context

Here's the exact sequence when execution hits an `await` on an incomplete task:

```
1. var awaiter = task.GetAwaiter();
2. if (!awaiter.IsCompleted)
   {
       // Capture the current SynchronizationContext
       var capturedCtx = SynchronizationContext.Current;
       // (if null, captures TaskScheduler.Current instead)
       
       // Register MoveNext as continuation
       // The builder posts MoveNext to capturedCtx when task completes
       builder.AwaitUnsafeOnCompleted(ref awaiter, ref stateMachine);
       return;  // suspend
   }
```

When the task completes later:

```
Task completes
    │
    ▼
Builder checks: was a SynchronizationContext captured?
    │                          │
   Yes                        No
    │                          │
    ▼                          ▼
capturedCtx.Post(            ThreadPool.QueueUserWorkItem(
    MoveNext, null);             MoveNext);
    │                          │
    ▼                          ▼
MoveNext runs on the         MoveNext runs on whatever
context's target thread      pool thread is available
(e.g., UI thread)
```

This is why in WPF/WinForms, you can touch UI controls after `await` — the continuation is **posted back** to the UI thread via the dispatcher context.


---

## ConfigureAwait(false) — Opting Out of the Context

`ConfigureAwait(false)` tells the await: "I don't need to resume on the captured context. Any thread pool thread is fine."

```csharp
var data = await DownloadAsync(url).ConfigureAwait(false);
// Resumes on ANY thread pool thread, NOT the UI thread
```

### What It Actually Returns

`ConfigureAwait(false)` returns a `ConfiguredTaskAwaitable` (not a `Task`). This is a wrapper struct whose `GetAwaiter()` returns a `ConfiguredTaskAwaitable.ConfiguredTaskAwaiter` — which behaves like a normal task awaiter but **skips capturing the `SynchronizationContext`**.

```csharp
// These are equivalent:
await task.ConfigureAwait(false);

// Conceptually:
var awaiter = task.ConfigureAwait(false).GetAwaiter();
// This awaiter does NOT post back to SynchronizationContext
```

### When to Use ConfigureAwait(false)

| Scenario | Use `ConfigureAwait(false)`? | Why |
|---|---|---|
| **Library code** (NuGet package, shared utility) | **Yes** | You don't know the caller's context. Posting back to a context you don't need wastes performance and risks deadlocks. |
| **Application code (UI)** | **No** (usually) | You likely need to touch UI controls after `await`. |
| **Application code (ASP.NET Core)** | **Doesn't matter** | There's no `SynchronizationContext` to capture, so `ConfigureAwait(false)` is a no-op. But it doesn't hurt. |
| **Application code (classic ASP.NET)** | **Yes** in non-controller code | Avoids the `AspNetSynchronizationContext` overhead. |

```ad-warning
The rule of thumb:
- **Library authors**: always use `ConfigureAwait(false)` on every `await` that doesn't need the context
- **Application developers**: omit it (default behavior) unless you have a specific reason

This asymmetry exists because library code doesn't know what context it'll be called from, while application code does.
```


---

## The Classic Deadlock: .Result + SynchronizationContext

This is the single most common async deadlock in C#, and it happens when you **synchronously block** on a task from a thread that owns a single-threaded `SynchronizationContext`.

```csharp
// In a WPF button click handler:
void Button_Click(object sender, RoutedEventArgs e)
{
    // DEADLOCK! ↓
    var result = GetDataAsync().Result;
    StatusLabel.Text = result;
}

async Task<string> GetDataAsync()
{
    var data = await HttpClient.GetStringAsync(url);
    // ↑ Captures DispatcherSynchronizationContext (UI thread)
    // When the HTTP call completes, it tries to post back to the UI thread
    // But the UI thread is BLOCKED on .Result above — it can't process the post
    return data;
}
```

Here's what happens step by step:

```
UI Thread                              Thread Pool / I/O
─────────                              ──────────────────
1. Button_Click calls GetDataAsync()
2. GetDataAsync starts, hits await
   Captures UI SynchronizationContext
3. HTTP request starts (I/O)
4. GetDataAsync returns incomplete Task
5. .Result BLOCKS the UI thread  ──┐
   (waiting for Task to complete)  │
                                   │    6. HTTP response arrives
                                   │    7. Continuation tries to Post
                                   │       to UI thread via context
                                   │    8. Post queues MoveNext to the
                                   │       dispatcher message loop
                                   │
   UI thread is blocked here ◄─────┘    9. Dispatcher can't run MoveNext
   waiting for Task.Result              because the UI thread is stuck
                                        waiting on .Result
   
   ╔══════════════════════════════════════════════════════════════╗
   ║  DEADLOCK: UI thread waits for Task, Task waits for UI      ║
   ║  thread. Neither can proceed.                                ║
   ╚══════════════════════════════════════════════════════════════╝
```

### Why This Only Happens with Single-Threaded Contexts

- **WPF / WinForms**: The `SynchronizationContext` targets **one specific thread** (the UI thread). If that thread is blocked, nothing can be posted to it. Deadlock.
- **ASP.NET Core / Console**: There is **no** `SynchronizationContext` (or it doesn't require a specific thread). Continuations just run on any pool thread, so blocking `.Result` doesn't deadlock — it just wastes a thread (thread pool starvation, which is still bad, but not a deadlock).

### The Fix

Option 1 — **Go async all the way** (preferred):

```csharp
async void Button_Click(object sender, RoutedEventArgs e)
{
    var result = await GetDataAsync();
    StatusLabel.Text = result;
}
```

Option 2 — **ConfigureAwait(false)** in the called method (if you don't need the context):

```csharp
async Task<string> GetDataAsync()
{
    var data = await HttpClient.GetStringAsync(url).ConfigureAwait(false);
    // Now resumes on a pool thread — doesn't need the UI thread
    // So .Result in the caller won't deadlock (but it's still not recommended)
    return data;
}
```

```ad-danger
Option 2 is a workaround, not a solution. Even if it avoids the deadlock, calling `.Result` on the UI thread **still freezes the UI** for the duration of the async operation. The correct fix is **always** Option 1: make the caller async.
```

See [[Common Async Pitfalls]] for more on the dangers of `.Result` and `.Wait()`.


---

## SynchronizationContext.Post vs Send

The `SynchronizationContext` class has two key methods:

| Method | Behavior |
|---|---|
| `Post(callback, state)` | **Asynchronous** — queues the callback and returns immediately. This is what `await` uses. |
| `Send(callback, state)` | **Synchronous** — blocks until the callback executes on the target thread. Rarely used by `await`. |

In UI contexts, `Post` adds to the message queue (like `BeginInvoke`), while `Send` blocks until the dispatcher processes it (like `Invoke`).

The `await` machinery always uses `Post` — it never blocks the completing thread waiting for the continuation to run.


---

## TaskScheduler as a Fallback

When `SynchronizationContext.Current` is `null`, the async infrastructure falls back to `TaskScheduler.Current`. In practice, this is almost always `TaskScheduler.Default`, which is the thread pool scheduler.

```
await continuation scheduling:

1. Check SynchronizationContext.Current
   │
   ├── Not null → Post continuation to that context
   │
   └── Null → Check TaskScheduler.Current
                │
                ├── Not Default → Schedule on that scheduler
                │
                └── Default → Queue to the thread pool
```

You rarely need to think about this. The only common case where `TaskScheduler.Current` matters is inside a `ContinueWith` callback that was scheduled on a custom `TaskScheduler` — and you should prefer `await` over `ContinueWith` anyway.


---

## ConfigureAwait in .NET 8+ and the Future

Starting in .NET 8, there's also `ConfigureAwait(ConfigureAwaitOptions)` with more granular control:

```csharp
// Suppress the context (same as ConfigureAwait(false))
await task.ConfigureAwait(ConfigureAwaitOptions.None);

// Don't throw on cancellation — the Task becomes canceled but await doesn't throw
await task.ConfigureAwait(ConfigureAwaitOptions.SuppressThrowing);

// Force continuation onto the thread pool (even if there's a context)
await task.ConfigureAwait(ConfigureAwaitOptions.ForceYielding);
```

| Option | Effect |
|---|---|
| `None` | Equivalent to `ConfigureAwait(false)` |
| `ContinueOnCapturedContext` | Equivalent to `ConfigureAwait(true)` (default) |
| `SuppressThrowing` | `await` returns normally even if the task is faulted/canceled |
| `ForceYielding` | Always yields — prevents synchronous continuation even when the task is already complete |

These can be combined with `|`:

```csharp
await task.ConfigureAwait(
    ConfigureAwaitOptions.ForceYielding | 
    ConfigureAwaitOptions.SuppressThrowing);
```

```ad-note
`ConfigureAwaitOptions` requires .NET 8+. If you're targeting older frameworks, stick with `ConfigureAwait(bool)`.
```


---

## See Also

- [[A First Look at the C# async and await Keywords]] — overview of async/await
- [[The Async State Machine]] — how the compiler generates the state machine
- [[Common Async Pitfalls]] — deadlocks, blocking, and other mistakes
- [[Exception Handling in Async Code]] — how exceptions interact with context
- [[The Task Class]] — Task fundamentals
- [[The CLR Thread Pool]] — where continuations execute when there's no context
