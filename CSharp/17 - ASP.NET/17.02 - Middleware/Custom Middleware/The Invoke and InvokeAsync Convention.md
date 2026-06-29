---
tags:
  - csharp
  - asp-net-core
  - middleware
  - custom
---

## The Invoke and InvokeAsync Convention

Convention-based middleware relies on a **naming convention** for its entry point method. ASP.NET Core looks for a method named either `Invoke` or `InvokeAsync` on the middleware class.

The rules:
1. The method must be `public`
2. It must return `Task` (not `void`, not `Task<T>`)
3. The first parameter must be `HttpContext`
4. Additional parameters are resolved from DI per request
5. You must have exactly **one** such method -- not both `Invoke` and `InvokeAsync`

```csharp
// Preferred: async version
public async Task InvokeAsync(HttpContext context)
{
    // Pre-processing
    await _next(context);
    // Post-processing
}

// Also valid: synchronous-looking but still returns Task
public Task Invoke(HttpContext context)
{
    if (SomeCondition(context))
    {
        context.Response.StatusCode = 403;
        return Task.CompletedTask; // Short-circuit without calling next
    }

    return _next(context); // Pass-through
}
```

> [!warning] Common Misconception
> Some developers define both `Invoke` and `InvokeAsync` on the same class, expecting one to be used for sync and the other for async scenarios. ASP.NET Core will throw an `InvalidOperationException` if both are present. Pick one.

> [!tip]
> Prefer `InvokeAsync` over `Invoke`. The `Async` suffix communicates the asynchronous nature of HTTP processing, and most middleware will use `await` internally.

> [!summary] Section Summary
> The middleware entry point must be a single public method named `Invoke` or `InvokeAsync`, returning `Task`, with `HttpContext` as its first parameter. Prefer `InvokeAsync`. Additional parameters beyond `HttpContext` are resolved from the DI container per request.
