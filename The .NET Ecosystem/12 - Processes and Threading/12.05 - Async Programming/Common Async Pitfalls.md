---
tags:
 - csharp
 - async
 - tpl
---

## Overview

Async/await in C# is designed to look like synchronous code, but the execution model is fundamentally different. This creates a class of bugs that are unique to asynchronous programming — most of them are **silent** (no compile error, no immediate crash) and only surface under specific conditions like load, UI interaction, or particular `SynchronizationContext` configurations.

This note covers the most common pitfalls, each with **wrong** and **right** examples.


---

## Pitfall 1: Calling .Result or .Wait() — Sync-Over-Async

**The problem**: Blocking on a Task with `.Result` or `.Wait()` from code that runs on a thread with a single-threaded `SynchronizationContext` causes a **deadlock**. Even without a context, it wastes a thread (thread pool starvation).

### The Deadlock Scenario

```csharp
// WRONG — deadlock in WPF/WinForms
void Button_Click(object sender, EventArgs e)
{
    var data = GetDataAsync().Result; // BLOCKS the UI thread
    Label.Text = data;
}

async Task<string> GetDataAsync()
{
    var response = await HttpClient.GetStringAsync(url);
    // ↑ Continuation needs the UI thread (captured SynchronizationContext)
    // But the UI thread is blocked on .Result above
    // DEADLOCK
    return response;
}
```

```
UI Thread:       .Result blocks ──────► waiting for Task to complete
                                            ▲
                                            │ circular dependency
                                            ▼
Continuation:    needs UI thread ──────► but UI thread is blocked
```

### Why It's Dangerous Even Without a Context

In ASP.NET Core or console apps (no `SynchronizationContext`), `.Result` won't deadlock, but it **blocks a thread pool thread**. Under load, this causes **thread pool starvation** — every blocked thread is one fewer available for actual work.

```csharp
// WRONG — no deadlock but wastes a thread pool thread
app.MapGet("/api/data", () =>
{
    var data = GetDataAsync().Result; // Blocks a pool thread for the entire I/O duration
    return data;
});
// Under 200 concurrent requests, you now have 200 blocked threads doing nothing
// See [[The CLR Thread Pool]] for why this matters
```

### The Fix

```csharp
// RIGHT — async all the way
async void Button_Click(object sender, EventArgs e)
{
    var data = await GetDataAsync();
    Label.Text = data;
}

// RIGHT — ASP.NET Core
app.MapGet("/api/data", async () =>
{
    var data = await GetDataAsync();
    return data;
});
```

```ad-warning
The **only** places where `.Result`/`.Wait()` are defensible:
- `Main()` in console apps (before C# 7.1 added `async Main`)
- Inside `Task.Run()` when you're already on a pool thread with no context and need to bridge sync/async
- After checking `task.IsCompletedSuccessfully` (the task is already done, so it won't block)

Even in these cases, prefer `await` or `GetAwaiter().GetResult()` (which unwraps the exception without `AggregateException` wrapping).
```

See [[SynchronizationContext and ConfigureAwait]] for the full deadlock mechanism and diagrams.


---

## Pitfall 2: Forgetting to await a Task

**The problem**: If you call an async method but don't `await` the returned Task, the task runs in the background with **no observation**. Exceptions are silently swallowed. The caller continues without waiting for completion.

```csharp
// WRONG — Task is not awaited
async Task ProcessOrderAsync(Order order)
{
    ValidateOrder(order);
    SaveOrderAsync(order);    // WARNING CS4014: this call is not awaited
    SendEmailAsync(order);    // WARNING CS4014: this call is not awaited
    // Method returns IMMEDIATELY — save and email haven't finished
    // If either throws, the exception is lost
}
```

The compiler gives you **CS4014**: "Because this call is not awaited, execution of the current method continues before the call is completed." This is a warning, not an error — it compiles and runs, but incorrectly.

### The Fix

```csharp
// RIGHT — await each task
async Task ProcessOrderAsync(Order order)
{
    ValidateOrder(order);
    await SaveOrderAsync(order);    // Wait for save
    await SendEmailAsync(order);    // Then wait for email
}

// RIGHT — or run them concurrently and await both
async Task ProcessOrderAsync(Order order)
{
    ValidateOrder(order);
    Task saveTask = SaveOrderAsync(order);
    Task emailTask = SendEmailAsync(order);
    await Task.WhenAll(saveTask, emailTask); // Both run concurrently, both observed
}
```

```ad-danger
**Never suppress CS4014 with `#pragma warning disable`.** It's telling you about a real bug. If you intentionally want fire-and-forget, make it explicit and handle exceptions:

```csharp
// Explicit fire-and-forget with error handling
_ = Task.Run(async () =>
{
    try
    {
        await SendEmailAsync(order);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Failed to send email for order {Id}", order.Id);
    }
});
```

The `_ =` discard makes the intent clear and suppresses CS4014.
```


---

## Pitfall 3: Not Going "Async All the Way"

**The problem**: If you make one method in a call chain async, but the callers above it are synchronous, you end up calling `.Result` or `.Wait()` somewhere — which brings back Pitfall 1.

```csharp
// WRONG — mixing sync and async in the call chain
class OrderService
{
    public OrderResult ProcessOrder(Order order)  // Sync method
    {
        var validated = ValidateOrder(order);
        var saved = SaveOrderAsync(validated).Result;  // Sync-over-async!
        return saved;
    }
}

class OrderController
{
    public IActionResult Post(Order order)  // Sync action
    {
        var result = _service.ProcessOrder(order);
        return Ok(result);
    }
}
```

### The Fix: Async All the Way Up

```csharp
// RIGHT — async from bottom to top
class OrderService
{
    public async Task<OrderResult> ProcessOrderAsync(Order order)
    {
        var validated = ValidateOrder(order);
        var saved = await SaveOrderAsync(validated);
        return saved;
    }
}

class OrderController
{
    public async Task<IActionResult> PostAsync(Order order)
    {
        var result = await _service.ProcessOrderAsync(order);
        return Ok(result);
    }
}
```

```
WRONG — sync-over-async:                  RIGHT — async all the way:

Controller.Post()         (sync)          Controller.PostAsync()      (async)
    │                                         │
    ▼                                         ▼ await
Service.ProcessOrder()    (sync)          Service.ProcessOrderAsync() (async)
    │                                         │
    ▼ .Result (BLOCKS!)                       ▼ await
Repository.SaveAsync()   (async)          Repository.SaveAsync()      (async)
    │                                         │
    ▼                                         ▼
Database I/O                              Database I/O
```

```ad-note
"Async all the way" means: **once you have one async method at the bottom, every caller up the chain should also be async.** The async keyword propagates upward through the call chain. This is sometimes called "async contagion" — it spreads through your codebase, and that's by design.
```


---

## Pitfall 4: Unnecessary async/await — Elision Bugs

**The problem**: Sometimes developers strip the `async`/`await` keywords to "avoid overhead" when the method just passes through to another async method. This is called **elision**, and it can introduce a subtle bug with resource disposal.

### When Elision Is Safe

```csharp
// Elided — no async/await keywords, just returns the Task directly
Task<string> GetDataAsync()
{
    return HttpClient.GetStringAsync(url);
    // No async keyword, no state machine generated
    // Slightly less overhead... but is it safe?
}
```

This is fine when there's **no `using`, `try`/`catch`, or `finally`** in the method.

### The Using Disposal Bug

```csharp
// WRONG — elision with using statement
Task<string> ReadFileAsync(string path)
{
    using var stream = new StreamReader(path);
    return stream.ReadToEndAsync();
    // stream.Dispose() is called HERE — when the method returns
    // But ReadToEndAsync hasn't finished yet!
    // The stream is disposed while still being read → ObjectDisposedException
}
```

```
Timeline:

ReadFileAsync() called
    │
    ├── StreamReader created
    ├── ReadToEndAsync() called → returns Task (reading in progress)
    ├── Method returns the Task
    ├── using scope ends → stream.Dispose() ← BUG: reading not done!
    │
    ▼
Task continues reading from a disposed stream → CRASH
```

### The Fix — Keep async/await When Using Resources

```csharp
// RIGHT — async/await keeps the method alive until the read completes
async Task<string> ReadFileAsync(string path)
{
    using var stream = new StreamReader(path);
    return await stream.ReadToEndAsync();
    // await suspends the state machine — stream is NOT disposed yet
    // Dispose happens AFTER the read completes
}
```

```ad-warning
**Rule**: Only elide `async`/`await` if the method has **no `using`, `try`/`catch`, or `finally` blocks** that need to stay alive for the duration of the async operation. When in doubt, keep `async`/`await` — the overhead of the state machine is negligible compared to any real I/O operation.
```

### When Elision Is Actually Worthwhile

```csharp
// Safe to elide — no resources, just forwarding
Task<User> GetUserAsync(int id)
    => _repository.FindAsync(id);

// Safe to elide — wrapping a completed value
Task<int> GetCachedCountAsync()
    => Task.FromResult(_cache.Count);
```


---

## Pitfall 5: async void in Non-Event-Handler Methods

**The problem**: Using `async void` anywhere except event handlers means exceptions crash the process and the caller can't await completion.

```csharp
// WRONG — async void for a regular method
async void ProcessInBackground(Order order)
{
    await SaveAsync(order);
    // If this throws, the process crashes
    // The caller can't await this or catch exceptions
}

// Caller has no idea when it finishes or if it failed:
ProcessInBackground(order);
Console.WriteLine("Done?"); // Printed immediately, not when processing finishes
```

### The Fix

```csharp
// RIGHT — return Task
async Task ProcessInBackgroundAsync(Order order)
{
    await SaveAsync(order);
}

// Now the caller can await and catch:
try
{
    await ProcessInBackgroundAsync(order);
    Console.WriteLine("Actually done");
}
catch (Exception ex)
{
    Console.WriteLine($"Failed: {ex.Message}");
}
```

See [[Async Return Types]] and [[Exception Handling in Async Code]] for the full details on `async void` dangers.


---

## Pitfall 6: Creating Tasks You Don't Need — Async Over Sync

**The problem**: Wrapping synchronous code in `Task.Run` inside a method and presenting it as async. This doesn't make the code genuinely asynchronous — it just moves synchronous work to a thread pool thread.

```csharp
// WRONG — fake async: synchronous code wrapped in Task.Run
async Task<int> ComputeAsync(int input)
{
    return await Task.Run(() =>
    {
        // This is CPU-bound synchronous work
        return HeavyComputation(input);
    });
}
```

This is problematic in **library code** because the caller doesn't know a thread pool thread is being consumed. The caller might already be on a pool thread, so you're now using **two** pool threads for work that only needs one.

```ad-warning
**The decision to offload to `Task.Run` belongs to the caller, not the library.** A library method should be honestly synchronous if the work is synchronous. The caller can wrap it in `Task.Run` if they want to offload it.
```

### The Fix

```csharp
// RIGHT — library exposes synchronous method honestly
int Compute(int input)
{
    return HeavyComputation(input);
}

// Caller decides whether to offload:
var result = await Task.Run(() => Compute(42));
```

The exception: **application code** (UI event handlers) where you know you're on the UI thread and need to move CPU work off it:

```csharp
// OK in UI code — intentionally offloading CPU work from the UI thread
async void Button_Click(object sender, EventArgs e)
{
    var result = await Task.Run(() => HeavyComputation(42));
    Label.Text = $"Result: {result}";
}
```


---

## Pitfall 7: Not Passing CancellationToken Through

**The problem**: Async methods often accept a `CancellationToken` parameter, but developers forget to pass it through to the inner async calls. The cancellation request never reaches the actual operation.

```csharp
// WRONG — CancellationToken is accepted but never used
async Task<string> FetchDataAsync(CancellationToken ct)
{
    var response = await HttpClient.GetAsync(url); // ct not passed!
    var body = await response.Content.ReadAsStringAsync(); // ct not passed!
    return body;
}
// Canceling the token does nothing — the HTTP request runs to completion
```

### The Fix

```csharp
// RIGHT — pass the token to every async call that accepts one
async Task<string> FetchDataAsync(CancellationToken ct)
{
    var response = await HttpClient.GetAsync(url, ct);
    var body = await response.Content.ReadAsStringAsync(ct);
    return body;
}
```

```ad-note
Also consider checking `ct.ThrowIfCancellationRequested()` between CPU-bound steps in long-running methods. Passing the token to I/O calls handles cancellation during I/O waits, but CPU-bound loops need explicit checks.
```


---

## Quick Reference: Wrong vs Right

| Pitfall | Wrong | Right |
|---|---|---|
| Sync-over-async | `task.Result`, `task.Wait()` | `await task` |
| Forgetting await | `SaveAsync(order);` | `await SaveAsync(order);` |
| Breaking the chain | sync caller → async callee → `.Result` | async all the way up |
| Elision with `using` | `return stream.ReadAsync()` (no await) | `return await stream.ReadAsync()` |
| async void misuse | `async void DoWork()` | `async Task DoWorkAsync()` |
| Fake async | `Task.Run(() => SyncWork())` in library | Expose sync method, let caller decide |
| Token not passed | `await GetAsync(url)` ignoring `ct` | `await GetAsync(url, ct)` |


---

## See Also

- [[A First Look at the C# async and await Keywords]] — overview of async/await
- [[SynchronizationContext and ConfigureAwait]] — the deadlock mechanism in detail
- [[Exception Handling in Async Code]] — what happens to swallowed exceptions
- [[Async Return Types]] — why async void is dangerous
- [[The Task Class]] — Task states, .Result, and .Wait()
- [[The CLR Thread Pool]] — thread pool starvation from sync-over-async
