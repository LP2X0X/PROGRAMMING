---
tags:
 - csharp
 - async
 - tpl
 - compiler
---

## What the `async` Keyword Actually Does

The `async` keyword does **not** make a method run on a background thread. It does **not** make anything parallel. It is a **compiler directive** that tells the C# compiler: "this method contains `await` expressions — transform it into a **state machine**."

That transformation is the entire point. The compiler rewrites your linear-looking code into a struct that can **suspend and resume** at each `await` point, without blocking a thread while waiting.

```csharp
// What you write:
async Task<int> GetDataAsync()
{
    var raw = await DownloadAsync("https://example.com");
    var parsed = await ParseAsync(raw);
    return parsed.Length;
}
```

The compiler sees two `await` points and generates a state machine with **three states**: before the first await, between the two awaits, and after the second await. Your local variables (`raw`, `parsed`) become **fields** on the state machine struct so they survive across suspensions.

```ad-warning
A common misconception: "`async` means the method runs asynchronously on another thread." No. `async` enables the compiler transformation. The method starts executing **synchronously on the calling thread** until it hits the first `await` on an incomplete task. If all awaited tasks are already complete, the entire method runs synchronously — no thread switching at all.
```


---

## The Generated State Machine

The compiler generates a struct that implements `IAsyncStateMachine`. Here's a **simplified** version of what the compiler produces for the method above:

```csharp
// Compiler-generated (simplified for clarity)
[CompilerGenerated]
private struct <GetDataAsync>d__0 : IAsyncStateMachine
{
    // State field: tracks which await we're at
    public int <>1__state;
    
    // Builder: manages the Task returned to the caller
    public AsyncTaskMethodBuilder<int> <>t__builder;
    
    // Hoisted locals — your local variables become fields
    private string <raw>5__1;
    private ParsedData <parsed>5__2;
    
    // Awaiters — one per distinct awaited type
    private TaskAwaiter<string> <>u__1;
    private TaskAwaiter<ParsedData> <>u__2;

    public void MoveNext()
    {
        int result;
        try
        {
            TaskAwaiter<string> awaiter1;
            TaskAwaiter<ParsedData> awaiter2;

            switch (<>1__state)
            {
                case -1:    // Initial state — first entry
                    goto State_0;
                case 0:     // Resuming after first await
                    awaiter1 = <>u__1;
                    goto Resume_0;
                case 1:     // Resuming after second await
                    awaiter2 = <>u__2;
                    goto Resume_1;
            }

        State_0:
            awaiter1 = DownloadAsync("https://example.com").GetAwaiter();
            if (!awaiter1.IsCompleted)
            {
                <>1__state = 0;
                <>u__1 = awaiter1;
                <>t__builder.AwaitUnsafeOnCompleted(ref awaiter1, ref this);
                return; // ← SUSPEND: method returns here, thread is free
            }

        Resume_0:
            <raw>5__1 = awaiter1.GetResult(); // get the result
            
            awaiter2 = ParseAsync(<raw>5__1).GetAwaiter();
            if (!awaiter2.IsCompleted)
            {
                <>1__state = 1;
                <>u__2 = awaiter2;
                <>t__builder.AwaitUnsafeOnCompleted(ref awaiter2, ref this);
                return; // ← SUSPEND again
            }

        Resume_1:
            <parsed>5__2 = awaiter2.GetResult();
            result = <parsed>5__2.Length;
        }
        catch (Exception ex)
        {
            <>1__state = -2; // Terminal state
            <>t__builder.SetException(ex);
            return;
        }

        <>1__state = -2; // Terminal state — completed
        <>t__builder.SetResult(result);
    }

    public void SetStateMachine(IAsyncStateMachine stateMachine)
    {
        <>t__builder.SetStateMachine(stateMachine);
    }
}
```

```ad-note
The angle brackets and numbers in names like `<>1__state` and `<raw>5__1` are how the compiler generates "unspeakable" names — names that are valid IL but invalid C#. This prevents collisions with your own code.
```


---

## Key Components of the State Machine

### The State Field (`<>1__state`)

This integer tracks where execution should resume. The convention is:

| Value | Meaning |
|---|---|
| `-1` | Initial entry — method hasn't started yet |
| `0` | Suspended at the first `await` |
| `1` | Suspended at the second `await` |
| `N` | Suspended at the (N+1)th `await` |
| `-2` | Terminal — method completed (success or exception) |

Each `await` in your code gets its own state number. **More awaits = more states = a larger switch in `MoveNext`**.

### The Builder (`AsyncTaskMethodBuilder<T>`)

The builder is responsible for:

1. **Creating the Task** that gets returned to the caller
2. **Managing continuations** — scheduling `MoveNext` to be called again when the awaited thing completes
3. **Setting the result** or **exception** on the Task when the method finishes

There are different builders for different return types:

| Return type | Builder |
|---|---|
| `Task` | `AsyncTaskMethodBuilder` |
| `Task<T>` | `AsyncTaskMethodBuilder<T>` |
| `ValueTask` | `AsyncValueTaskMethodBuilder` |
| `ValueTask<T>` | `AsyncValueTaskMethodBuilder<T>` |
| `async void` | `AsyncVoidMethodBuilder` |

See [[Async Return Types]] for when to use each.

### Hoisted Local Variables

Every local variable that is alive across an `await` boundary becomes a **field** on the state machine struct. This is necessary because when the method suspends, the stack frame is gone — the thread returns to the pool. The only way to preserve the local state is to store it on the heap (as fields of the struct, which gets boxed to the heap when the first suspension occurs).

```csharp
async Task ExampleAsync()
{
    int x = 10;                    // hoisted — lives across the await
    var data = await FetchAsync(); // ← suspension point
    Console.WriteLine(x + data);   // x must still be available here
    
    int y = 20;                    // NOT hoisted if nothing awaits after this
    Console.WriteLine(y);
}
```

```ad-note
The compiler is smart about this. Variables that don't span an `await` boundary may remain true locals inside `MoveNext` and never become fields. The JIT can even keep them in registers.
```


---

## How Suspension and Resumption Work

Here's the full flow of what happens at an `await`:

```
Your code:   var result = await SomeAsync();
                  │
                  ▼
    ┌─────────────────────────────────┐
    │ 1. Call SomeAsync()             │ ← Synchronous: returns a Task
    │    Gets back a Task<T>          │
    └──────────────┬──────────────────┘
                   │
                   ▼
    ┌─────────────────────────────────┐
    │ 2. Call task.GetAwaiter()       │
    │    Gets a TaskAwaiter<T>        │
    └──────────────┬──────────────────┘
                   │
                   ▼
    ┌─────────────────────────────────┐
    │ 3. Check awaiter.IsCompleted    │
    └──────┬───────────────┬──────────┘
           │               │
      Already done     Not done yet
           │               │
           ▼               ▼
    ┌─────────────┐  ┌──────────────────────────────────┐
    │ GetResult() │  │ 4. Save state number              │
    │ Continue    │  │    Store awaiter in field          │
    │ synchronous │  │    builder.AwaitOnCompleted(...)   │
    │ execution   │  │    ─── registers MoveNext as the  │
    │             │  │        continuation on the Task    │
    └─────────────┘  │ 5. RETURN from MoveNext           │
                     │    ─── thread is FREE              │
                     └──────────────┬───────────────────┘
                                    │
                           (time passes, I/O completes)
                                    │
                                    ▼
                     ┌──────────────────────────────────┐
                     │ 6. Task completes → continuation  │
                     │    fires → MoveNext called again  │
                     │    Switch lands on saved state     │
                     │    GetResult() retrieves value     │
                     │    Execution continues from there  │
                     └──────────────────────────────────┘
```

The critical insight is step 5: **the method literally returns**. The calling thread is freed. No thread is blocked waiting. When the I/O completes, the continuation calls `MoveNext` again — potentially on a **different thread** (unless [[SynchronizationContext and ConfigureAwait|SynchronizationContext]] captures it back to the original thread).


---

## Why `ref` Locals and `ref struct` Can't Span an `await`

Because local variables get hoisted to fields on the state machine struct, anything that **cannot be a field** cannot survive an `await` boundary.

**`ref` locals** are pointers into the stack. When the method suspends at an `await`, the stack frame is destroyed. A `ref` local pointing into that destroyed frame would be a dangling pointer.

**`ref struct` types** (like `Span<T>` and `ReadOnlySpan<T>`) can only live on the stack — they cannot be heap-allocated, which means they cannot be fields on the state machine struct.

```csharp
async Task ProcessAsync(byte[] buffer)
{
    Span<byte> span = buffer.AsSpan(); // ← Span is a ref struct
    await Task.Delay(1000);            // ← COMPILE ERROR!
    // CS4012: Parameters or locals of type 'Span<byte>' cannot be
    // declared in async methods or async lambda expressions.
    ProcessSpan(span);
}
```

The fix: scope the `Span<T>` so it doesn't cross the `await`:

```csharp
async Task ProcessAsync(byte[] buffer)
{
    await Task.Delay(1000);
    
    // Span is created and consumed BETWEEN awaits — never crosses one
    Span<byte> span = buffer.AsSpan();
    ProcessSpan(span);
}
```

```ad-note
Starting with C# 13 and .NET 9, the compiler has become smarter about `ref struct` in async methods. You can declare a `ref struct` local in an async method **as long as it does not live across an `await`**. Earlier compiler versions rejected `ref struct` locals in async methods entirely, even if they didn't cross an `await`.
```


---

## Mapping Your Code to States

Every `await` expression in the original method creates a **state boundary**. Here's how to visualize which parts of your code map to which states:

```
async Task<string> ProcessAsync(int id)        STATE -1 (initial)
{                                               │
    Console.WriteLine("Starting");              │  Runs synchronously
    var url = BuildUrl(id);                     │  on first MoveNext call
                                                │
    var raw = await HttpClient.GetAsync(url);   ◄── Suspension point 0
                                                │
    Console.WriteLine("Downloaded");            │  STATE 0 → resumes here
    var parsed = Parse(raw);                    │
                                                │
    await SaveToDatabaseAsync(parsed);          ◄── Suspension point 1
                                                │
    Console.WriteLine("Saved");                 │  STATE 1 → resumes here
    return parsed.Summary;                      │
}                                               STATE -2 (terminal)
```

| State | Code region | Triggered by |
|---|---|---|
| `-1` (initial) | From method start to first `await` | First call to `MoveNext` |
| `0` | Between first and second `await` | First task completes |
| `1` | From second `await` to method end | Second task completes |
| `-2` (terminal) | N/A — method has finished | Builder sets result/exception |

```ad-note
If the awaited task is **already completed** (e.g., `Task.FromResult`, cached result, or a truly fast I/O), `IsCompleted` is `true` and the method **does not suspend** — it falls through to the next section synchronously. The state field is never set to that state number. This is the **fast path** and it's a significant performance optimization.
```


---

## Performance Considerations

The state machine is initially a **struct** (`valuetype`). As long as the method completes synchronously (all awaited tasks already complete), it stays on the stack — no heap allocation at all. This is why `ValueTask` exists: for hot paths where completion is synchronous most of the time. See [[Async Return Types]].

However, the **first time the method actually suspends** (an awaited task is not yet complete), the struct must be **boxed to the heap**. The continuation needs to survive beyond the current stack frame, so the state machine moves to the managed heap. This is a one-time allocation per async call that actually suspends.

```
Synchronous completion (fast path):
  State machine stays as struct on the stack → zero allocation

First actual suspension:
  State machine gets boxed to the heap → one allocation
  All subsequent suspensions reuse the same boxed instance
```

The `AsyncTaskMethodBuilder` is designed to make the synchronous-completion path as cheap as possible. The pool of cached `Task<bool>` results for `true` and `false`, and `Task.CompletedTask` for non-generic `Task`, all serve this goal.

```ad-note
Don't prematurely optimize async code by trying to avoid the state machine. The overhead of one heap allocation per truly-async call is almost always negligible compared to the I/O operation you're awaiting. Focus on avoiding unnecessary async wrappers and [[Common Async Pitfalls|common pitfalls]] instead.
```


---

## See Also

- [[A First Look at the C# async and await Keywords]] — overview of async/await
- [[SynchronizationContext and ConfigureAwait]] — what thread resumes after `await`
- [[Async Return Types]] — Task, ValueTask, async void
- [[The Task Class]] — the underlying Task abstraction
- [[The CLR Thread Pool]] — where continuations execute
