---
tags:
 - csharp
 - async
 - tpl
 - exceptions
---

## How Exceptions Work in Async Methods

When an exception is thrown inside an `async Task` or `async Task<T>` method, it does **not** propagate up the call stack immediately. Instead, the exception is **captured and stored on the Task** object. The Task transitions to the `Faulted` state, and the exception waits there until someone observes it.

```csharp
async Task<int> FaultyAsync()
{
    await Task.Delay(100);
    throw new InvalidOperationException("something broke");
    // Exception is captured → Task becomes Faulted
    // The exception does NOT fly up the call stack here
}

Task<int> task = FaultyAsync();
// At this point, the Task is running (or already faulted)
// No exception has been thrown to *this* code yet
```

The exception only surfaces when someone **observes** the task — via `await`, `.Result`, `.Wait()`, or `.Exception`.

This is fundamentally different from synchronous code, where a `throw` immediately unwinds the stack.


---

## await Rethrows the Exception — try/catch Works Naturally

The most important thing to know: **`await` re-throws the captured exception**. This means `try`/`catch` around an `await` works exactly like you'd expect from synchronous code.

```csharp
try
{
    int result = await FaultyAsync();
    Console.WriteLine(result); // never reached
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"Caught: {ex.Message}");
    // "Caught: something broke"
}
```

This is one of the key design goals of async/await: **exception handling looks and feels like synchronous code**. No special patterns, no unwrapping — just `try`/`catch`.

Behind the scenes, the [[The Async State Machine|state machine's]] `MoveNext` method catches the exception from `GetResult()` and lets it propagate normally into your `catch` block.

```ad-note
`await` specifically re-throws the **first exception** stored on the Task, not an `AggregateException`. This is an intentional design choice to make async exception handling feel natural. The original exception's stack trace is preserved.
```


---

## await vs .Result/.Wait() — AggregateException Wrapping

Here's where the behavior diverges. `.Result` and `.Wait()` were designed before `await` existed (they're part of TPL, introduced in .NET 4.0). They wrap exceptions in an `AggregateException`:

| Observation method | Exception behavior |
|---|---|
| `await task` | Re-throws the **first inner exception** directly |
| `task.Result` | Throws **`AggregateException`** containing all inner exceptions |
| `task.Wait()` | Throws **`AggregateException`** containing all inner exceptions |
| `task.GetAwaiter().GetResult()` | Re-throws the **first inner exception** directly (same as `await`) |

```csharp
async Task ThrowsAsync()
{
    throw new InvalidOperationException("broke");
}

// With await — catches the actual exception:
try
{
    await ThrowsAsync();
}
catch (InvalidOperationException ex)
{
    Console.WriteLine(ex.Message); // "broke"
}

// With .Result — catches AggregateException:
try
{
    ThrowsAsync().Result; // BLOCKS — and wraps in AggregateException
}
catch (AggregateException agg)
{
    // Must dig in to get the real exception
    Console.WriteLine(agg.InnerException!.Message); // "broke"
}
catch (InvalidOperationException ex)
{
    // This catch block is NEVER reached!
}
```

```ad-warning
This is a common source of confusion. If you catch `InvalidOperationException` but the code uses `.Result` instead of `await`, the catch block won't trigger because `.Result` throws `AggregateException`. If you must use `.Result`, either catch `AggregateException` or use `.GetAwaiter().GetResult()` instead (which unwraps like `await`).
```


---

## Task.WhenAll and Multiple Exceptions

`Task.WhenAll` waits for **all** tasks to complete, even if some fail. If multiple tasks throw, all exceptions are collected — but **`await` only re-throws the first one**.

```csharp
var task1 = Task.Run(() => throw new InvalidOperationException("error 1"));
var task2 = Task.Run(() => throw new ArgumentException("error 2"));
var task3 = Task.Run(() => throw new TimeoutException("error 3"));

Task allTasks = Task.WhenAll(task1, task2, task3);

try
{
    await allTasks;
}
catch (InvalidOperationException ex)
{
    // Only the FIRST exception is thrown by await
    Console.WriteLine($"Caught: {ex.Message}"); // "error 1"
}
```

### Getting All Exceptions from WhenAll

To see **all** exceptions, inspect the `Task.Exception` property:

```csharp
Task allTasks = Task.WhenAll(task1, task2, task3);

try
{
    await allTasks;
}
catch
{
    // allTasks.Exception is an AggregateException containing ALL exceptions
    AggregateException all = allTasks.Exception!;
    
    foreach (var ex in all.InnerExceptions)
    {
        Console.WriteLine($"  {ex.GetType().Name}: {ex.Message}");
    }
    // Output:
    //   InvalidOperationException: error 1
    //   ArgumentException: error 2
    //   TimeoutException: error 3
}
```

A common pattern to handle all exceptions:

```csharp
Task allTasks = Task.WhenAll(task1, task2, task3);

try
{
    await allTasks;
}
catch (Exception ex) when (allTasks.Exception is { } agg)
{
    // Log or handle all failures
    foreach (var inner in agg.InnerExceptions)
    {
        _logger.LogError(inner, "Task failed");
    }
}
```

```ad-note
The order of `InnerExceptions` matches the order you passed the tasks to `WhenAll`. `await` always re-throws the first one — but "first" means the first task in the parameter list that faulted, not necessarily the one that faulted first in wall-clock time.
```


---

## Task.WhenAll with Task\<T> — Results and Exceptions

When using `Task.WhenAll` with `Task<T>` (tasks that return values), you have to decide how to handle partial results:

```csharp
Task<int> t1 = Task.Run(() => 10);
Task<int> t2 = Task.Run<int>(() => throw new Exception("failed"));
Task<int> t3 = Task.Run(() => 30);

try
{
    int[] results = await Task.WhenAll(t1, t2, t3);
    // Never reached because t2 faults
}
catch
{
    // But t1 and t3 completed successfully — their results are available:
    if (t1.Status == TaskStatus.RanToCompletion)
        Console.WriteLine($"t1 result: {t1.Result}"); // 10
    
    if (t3.Status == TaskStatus.RanToCompletion)
        Console.WriteLine($"t3 result: {t3.Result}"); // 30
}
```


---

## async void Exception Behavior

Exceptions in `async void` methods are **not captured on a Task** — there is no Task. Instead, the `AsyncVoidMethodBuilder` raises the exception on the `SynchronizationContext` that was current when the method started.

```csharp
async void FireAndForget()
{
    await Task.Delay(100);
    throw new Exception("async void exception");
    // This exception is raised on the SynchronizationContext
    // In most cases, this crashes the process
}
```

What happens depends on the context:

| Context | async void exception behavior |
|---|---|
| **WPF / WinForms** | Raised on the UI thread via `Dispatcher.UnhandledException` / `Application.ThreadException`. Can be caught at the app level, but the default is process crash. |
| **ASP.NET Core** | Raised on a thread pool thread. Unhandled = process crash. |
| **Console app** | Raised on a thread pool thread. Unhandled = process crash (`AppDomain.UnhandledException`). |
| **xUnit** | Depends on the SynchronizationContext the test framework installs. Often silently swallowed, causing the test to pass when it should fail. |

```ad-danger
The caller of an `async void` method has **no way** to catch its exceptions. This code does NOT work:

```csharp
try
{
    FireAndForget(); // Returns immediately (void, not Task)
}
catch (Exception ex)
{
    // NEVER reached — the exception happens later,
    // on a different stack, after this try/catch is gone
}
```

This is why `async void` should **only** be used for event handlers, and those handlers should **always** have a try/catch wrapping the entire body.
```

See [[Async Return Types]] for more on why async void is dangerous.


---

## Exceptions Before the First await

An important nuance: exceptions thrown **before the first `await`** in an `async Task` method are still captured on the Task, not thrown synchronously.

```csharp
async Task ValidateAndProcessAsync(string input)
{
    if (input == null)
        throw new ArgumentNullException(nameof(input)); // Before any await
    
    await ProcessAsync(input);
}

// The ArgumentNullException is NOT thrown here:
Task task = ValidateAndProcessAsync(null);
// It's captured on the task. This line runs fine.

// The exception surfaces here:
await task; // throws ArgumentNullException
```

This is different from how `Task.Run` behaves if you throw before it even starts. The `async` state machine **always** captures exceptions on the returned Task, regardless of where in the method they occur.

```ad-note
Some developers prefer to validate parameters eagerly (throw before returning the Task) by splitting the method:

```csharp
// Eager validation — exception thrown synchronously to the caller
Task ProcessAsync(string input)
{
    if (input == null)
        throw new ArgumentNullException(nameof(input)); // Thrown NOW
    
    return ProcessCoreAsync(input);
}

private async Task ProcessCoreAsync(string input)
{
    await DoWorkAsync(input);
}
```

This pattern is used in the .NET runtime libraries. The trade-off is more code but earlier error detection — the caller sees the `ArgumentNullException` at the call site, not at the `await` site.
```


---

## Unobserved Task Exceptions

What happens if a Task faults but nobody ever `await`s it, checks `.Result`, or inspects `.Exception`? The exception is **unobserved**.

In .NET 4.0, unobserved task exceptions would **crash the process** when the Task was garbage collected. Starting with .NET 4.5, the default changed to **swallowing** unobserved exceptions (they're silently ignored).

You can still detect them via the `TaskScheduler.UnobservedTaskException` event:

```csharp
TaskScheduler.UnobservedTaskException += (sender, e) =>
{
    _logger.LogError(e.Exception, "Unobserved task exception");
    e.SetObserved(); // Prevent the exception from being rethrown
};
```

```ad-warning
Unobserved exceptions are a sign of a bug — it means a Task faulted and nobody checked. Common causes:
- Forgetting to `await` a Task (see [[Common Async Pitfalls]])
- Fire-and-forget calls without exception handling
- Abandoned `ContinueWith` chains

The `UnobservedTaskException` event fires during **garbage collection** (on the finalizer thread), which can be long after the actual exception occurred. Don't rely on it for real-time error handling.
```


---

## Cancellation vs Exceptions

`CancellationToken` and `OperationCanceledException` have special handling in the async world:

```csharp
async Task DoWorkAsync(CancellationToken ct)
{
    await Task.Delay(5000, ct); // Throws OperationCanceledException if canceled
}

var cts = new CancellationTokenSource();
cts.CancelAfter(TimeSpan.FromSeconds(1));

try
{
    await DoWorkAsync(cts.Token);
}
catch (OperationCanceledException)
{
    Console.WriteLine("Operation was canceled");
    // Task.Status is TaskStatus.Canceled (not Faulted)
}
```

| Exception type | Task state | `IsCanceled` | `IsFaulted` |
|---|---|---|---|
| `OperationCanceledException` (matching token) | `Canceled` | `true` | `false` |
| `OperationCanceledException` (no/wrong token) | `Faulted` | `false` | `true` |
| Any other exception | `Faulted` | `false` | `true` |

```ad-note
The distinction between `Canceled` and `Faulted` matters for `Task.WhenAll`, retry logic, and logging. A canceled task is expected (the caller requested cancellation), while a faulted task is an error.
```


---

## Summary: Exception Observation Methods

```
             async Task method throws
                      │
              ┌───────┴───────┐
              ▼               ▼
        Exception is     Exception is
        captured on      NOT on a Task
        the Task         (async void)
              │               │
              │               ▼
              │         Posted to
              │         SynchronizationContext
              │         (usually crashes)
              │
    How you observe it:
              │
     ┌────────┼────────────┬──────────────┐
     ▼        ▼            ▼              ▼
   await    .Result     .Wait()     .Exception
     │        │            │              │
     ▼        ▼            ▼              ▼
  Original  Aggregate   Aggregate   AggregateException
  exception Exception   Exception   (inspect, don't
  directly  (wrapped)   (wrapped)    throw)
```


---

## See Also

- [[A First Look at the C# async and await Keywords]] — overview of async/await
- [[Async Return Types]] — how exception behavior differs by return type
- [[Common Async Pitfalls]] — forgetting to await and other exception-related mistakes
- [[The Async State Machine]] — how the state machine captures exceptions
- [[The Task Class]] — Task states and exception properties
