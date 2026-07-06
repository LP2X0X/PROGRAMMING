---
tags:
 - csharp
 - async
 - tpl
---

## Overview of Async Return Types

An `async` method must return one of a small set of types. The return type tells the caller **how to observe the result** (or completion) of the asynchronous operation. Choosing the right one affects performance, exception propagation, and API design.

| Return type | Use when | Returns a value? | Awaitable? |
|---|---|---|---|
| `Task<T>` | Method produces a result of type `T` | Yes | Yes |
| `Task` | Method completes but produces no result | No | Yes |
| `ValueTask<T>` | Hot path, often completes synchronously, returns `T` | Yes | Yes |
| `ValueTask` | Hot path, often completes synchronously, no result | No | Yes |
| `async void` | Event handlers only | No | **No** |
| `IAsyncEnumerable<T>` | Produces a stream of values asynchronously | Yes (stream) | `await foreach` |

The mapping from synchronous patterns:

| Synchronous | Async equivalent |
|---|---|
| `T MethodName()` | `Task<T> MethodNameAsync()` |
| `void MethodName()` | `Task MethodNameAsync()` |
| `void OnEvent(...)` (event handler) | `async void OnEvent(...)` |
| `IEnumerable<T> GetItems()` | `IAsyncEnumerable<T> GetItemsAsync()` |


---

## Task\<T> and Task — The Defaults

`Task<T>` and `Task` are the **default choices** for async return types. Unless you have a specific reason to use something else, use these.

### Task\<T> — Async Method That Returns a Value

```csharp
async Task<int> ComputeAsync()
{
    var data = await FetchDataAsync();
    return data.Length; // The int gets wrapped in the Task<int>
}

// Caller:
int length = await ComputeAsync();
```

The compiler wraps your `return` value into the `Task<T>` via the `AsyncTaskMethodBuilder<T>`. The caller gets a `Task<int>` immediately (before the method completes), and `await` unwraps the `int` when it's ready.

### Task — Async Method with No Return Value

```csharp
async Task SaveAsync(string data)
{
    await File.WriteAllTextAsync("output.txt", data);
    // No return statement needed — Task (not Task<T>) signals completion only
}

// Caller:
await SaveAsync("hello");
```

`Task` is the async equivalent of `void` — it represents an operation that completes but doesn't produce a value. The caller can still `await` it to know when it's done and to observe any exceptions.

### When the Method Completes Synchronously

If all `await` expressions in the method complete synchronously (the tasks are already done), the runtime can optimize:

- For `Task<bool>` returning `true` or `false`, the runtime uses **cached singleton tasks** — no allocation at all
- For `Task` returning successfully, `Task.CompletedTask` is reused
- For other `Task<T>` values, a new `Task<T>` object is allocated on the heap

This is where `ValueTask<T>` comes in — it avoids even that allocation.


---

## ValueTask\<T> and ValueTask — The Performance Option

**`ValueTask<T>`** is a struct (value type) that can represent **either** a completed result (`T` value) **or** an incomplete `Task<T>`. It's designed for methods that **frequently complete synchronously** where the allocation of a `Task<T>` object on the heap is wasteful.

```csharp
// A cache lookup — hits the cache most of the time (synchronous),
// only goes to the database on a miss (asynchronous)
async ValueTask<User> GetUserAsync(int id)
{
    if (_cache.TryGetValue(id, out User cached))
        return cached;  // Synchronous: no Task allocated, just wraps the value
    
    var user = await _database.LoadUserAsync(id); // Async path: wraps a Task
    _cache[id] = user;
    return user;
}
```

### How ValueTask Saves Allocations

```
Task<T> — always allocates a Task object on the heap:
  ┌─────────────────────────────┐
  │ Task<User> (heap object)    │
  │   .Result = cached user     │  ← allocated even for a simple cache hit
  │   .Status = RanToCompletion │
  └─────────────────────────────┘

ValueTask<T> — synchronous path = no heap allocation:
  ┌─────────────────────────────┐
  │ ValueTask<User> (stack)     │
  │   ._value = cached user     │  ← just a struct on the stack
  │   ._task  = null            │     no heap allocation at all
  └─────────────────────────────┘

ValueTask<T> — async path = wraps a Task internally:
  ┌─────────────────────────────┐
  │ ValueTask<User> (stack)     │
  │   ._value = default         │
  │   ._task  = ──────────────────► Task<User> (heap, from the async path)
  └─────────────────────────────┘
```

### When to Use ValueTask vs Task

| Factor | Use `Task<T>` | Use `ValueTask<T>` |
|---|---|---|
| Completion pattern | Usually async | Usually sync (cache, buffer, pre-fetched) |
| Consumed by | Any code | Disciplined callers (see restrictions below) |
| Public API | Safe default | Use if profiling shows allocation pressure |
| Simplicity | Simpler — no restrictions | More restrictions on usage |

```ad-warning
**Don't default to `ValueTask` everywhere.** It adds complexity and has restrictions. Use `Task<T>` unless you have **measured evidence** that the allocation matters on that specific hot path. Premature optimization with `ValueTask` is a common mistake.
```

### Critical Restrictions on ValueTask

A `ValueTask<T>` is **not** a general-purpose replacement for `Task<T>`. It has strict usage rules:

1. **Await it only once.** You cannot `await` the same `ValueTask<T>` twice.
2. **Don't await it concurrently.** Two threads cannot `await` the same `ValueTask<T>`.
3. **Don't use `.GetAwaiter().GetResult()` before it completes** (unless you know it's done).
4. **Don't call `AsTask()` more than once.**

```csharp
// WRONG — awaiting a ValueTask twice
ValueTask<int> vt = ComputeAsync();
int a = await vt;
int b = await vt;  // BUG: undefined behavior, may throw, may return garbage

// WRONG — concurrent await
ValueTask<int> vt = ComputeAsync();
Task.WhenAll(WrapAsync(vt), WrapAsync(vt)); // Two consumers = BUG

// RIGHT — if you need multiple consumers, convert to Task first
ValueTask<int> vt = ComputeAsync();
Task<int> task = vt.AsTask(); // Now it's a regular Task — safe to share
int a = await task;
int b = await task; // Fine — Task supports multiple awaits
```

```ad-danger
Violating these rules doesn't always crash. The behavior is **undefined** — it may silently return wrong values, throw random exceptions, or work today and break tomorrow. The restrictions exist because `ValueTask<T>` may be backed by a pooled `IValueTaskSource` that gets returned to the pool after the first `await`.
```

### Why These Restrictions Exist

`ValueTask<T>` can be backed by a pooled `IValueTaskSource<T>` object. After you `await` it, the backing object is **returned to the pool** for reuse. A second `await` would access a recycled object that's now serving a different operation.

```
First await:
  ValueTask<int> → IValueTaskSource<int> (object #7 from the pool)
  await completes → GetResult() returns 42
  object #7 is returned to the pool ✓

Second await (BUG):
  ValueTask<int> → same IValueTaskSource<int>... but object #7 is now
  serving a completely different async operation
  GetResult() returns ??? (undefined behavior)
```


---

## async void — The Dangerous One

`async void` is a valid async return type, but it is **fundamentally different** from `async Task`:

| | `async Task` | `async void` |
|---|---|---|
| Awaitable | Yes | **No** |
| Exceptions | Captured on the Task, rethrown by `await` | **Posted to `SynchronizationContext` — crashes the process in most cases** |
| Caller knows when it finishes | Yes, via `await` | **No** |
| Testable | Yes | **Very difficult** |

```csharp
// DANGEROUS — cannot be awaited, exceptions crash the process
async void DeleteEverythingAsync()
{
    await Task.Delay(1000);
    throw new Exception("oops");
    // This exception has NOWHERE to go — it crashes the process
}

// The caller has no way to:
DeleteEverythingAsync();
// - know when it finishes
// - catch its exceptions
// - await it
```

### The One Legitimate Use: Event Handlers

Event handler signatures are dictated by the event — they must return `void`. This is the **only** place where `async void` is acceptable:

```csharp
// This is OK — event handler signature requires void
async void Button_Click(object sender, EventArgs e)
{
    try
    {
        StatusLabel.Text = "Working...";
        var result = await DoWorkAsync();
        StatusLabel.Text = $"Done: {result}";
    }
    catch (Exception ex)
    {
        // IMPORTANT: catch everything here — exceptions in async void
        // that escape the method will crash the process
        StatusLabel.Text = $"Error: {ex.Message}";
    }
}
```

```ad-danger
If you use `async void` for an event handler, **always wrap the entire body in a try/catch**. An unhandled exception in `async void` is posted to the `SynchronizationContext`, which typically raises it as an unobserved exception that terminates the process.
```

### Why async void Exceptions Crash

An `async Task` method captures exceptions on the `Task` object. The caller observes them via `await`, `.Result`, or `.Exception`. The exception is contained.

An `async void` method has **no Task**. There's no container for the exception. So the `AsyncVoidMethodBuilder` posts it to the `SynchronizationContext` using `Post(callback_that_throws, null)`. In most contexts, this means the exception is re-raised on the original thread (UI thread in WPF/WinForms) or on a thread pool thread (console apps), and if nobody catches it there, the process crashes.

See [[Exception Handling in Async Code]] for a deeper look at exception flows.


---

## IAsyncEnumerable\<T> — Async Streams

`IAsyncEnumerable<T>` is the async version of `IEnumerable<T>`. It produces a **stream of values** where each value may require an asynchronous operation to produce.

```csharp
async IAsyncEnumerable<int> GenerateNumbersAsync(int count)
{
    for (int i = 0; i < count; i++)
    {
        await Task.Delay(100); // simulate async work per item
        yield return i;
    }
}

// Consumer:
await foreach (var number in GenerateNumbersAsync(10))
{
    Console.WriteLine(number);
}
```

The compiler generates a state machine for `async IAsyncEnumerable<T>` methods just like for regular `async Task` methods, but with the added complexity of `yield return` semantics. See [[The Async State Machine]] for how the state machine works.

```ad-note
`IAsyncEnumerable<T>` was introduced in C# 8 / .NET Core 3.0. It supports cancellation via `[EnumeratorCancellation] CancellationToken` parameter and `WithCancellation()` extension method.
```

```csharp
async IAsyncEnumerable<string> ReadLinesAsync(
    string path,
    [EnumeratorCancellation] CancellationToken ct = default)
{
    using var reader = new StreamReader(path);
    while (await reader.ReadLineAsync(ct) is { } line)
    {
        yield return line;
    }
}

// Cancel after 5 seconds:
var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
await foreach (var line in ReadLinesAsync("huge.txt").WithCancellation(cts.Token))
{
    Console.WriteLine(line);
}
```


---

## Naming Convention: The Async Suffix

By convention, all async methods that return `Task`, `Task<T>`, `ValueTask`, or `ValueTask<T>` should have the **`Async` suffix**:

```csharp
// ✓ Correct naming
Task<User> GetUserAsync(int id);
Task SaveAsync(string data);
ValueTask<byte[]> ReadBufferAsync();

// ✗ Missing suffix — confusing for callers
Task<User> GetUser(int id);    // Is this sync or async?
```

The exceptions:

- **Event handlers** (`async void`) don't use the suffix — they follow the event naming convention (`Button_Click`, `OnLoaded`, etc.)
- **Interface implementations** where the interface already defines the name (e.g., `IDisposable.Dispose` vs `IAsyncDisposable.DisposeAsync` — the framework already handles the naming)
- **Top-level APIs** where the entire API is async and the suffix would be noise (some newer .NET APIs drop it, but this is debated)

```ad-note
The suffix is a convention, not a compiler requirement. The compiler doesn't care. But other developers do — the suffix is a critical signal that the method returns a task that should be `await`ed.
```


---

## See Also

- [[A First Look at the C# async and await Keywords]] — overview of async/await
- [[The Async State Machine]] — compiler transformation and builders
- [[Exception Handling in Async Code]] — how exceptions differ by return type
- [[Common Async Pitfalls]] — async void misuse and other mistakes
- [[The Task Class]] — the Task fundamentals
