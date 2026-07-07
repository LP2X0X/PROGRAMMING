---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
  - request
---

## The Pipeline Model

The ASP.NET Core request pipeline is a **chain of delegates** -- each middleware component is a function that receives the `HttpContext` and a reference to the `next` delegate. When a request arrives, it flows through the chain from top to bottom. When a response is generated, it flows back through the chain from bottom to top.

```
Request -->  [Middleware 1]  -->  [Middleware 2]  -->  [Middleware 3]  -->  Endpoint
             <--                 <--                 <--                 Response
```

Each middleware component can:

1. **Do work before** calling `next()` (request processing)
2. **Call `next()`** to pass the request to the next middleware
3. **Do work after** `next()` returns (response processing)
4. **Skip calling `next()`** entirely (short-circuit the pipeline)

```csharp
app.Use(async (context, next) =>
{
    // 1. Work BEFORE the next middleware (request phase)
    var stopwatch = Stopwatch.StartNew();

    await next(context); // 2. Pass to the next middleware

    // 3. Work AFTER the next middleware returns (response phase)
    stopwatch.Stop();
    Console.WriteLine($"Request took {stopwatch.ElapsedMilliseconds}ms");
});
```

> [!summary] Section Summary
> The pipeline is a chain of middleware delegates. Each middleware can process the request, pass it along via `next()`, and optionally process the response on the way back. Skipping `next()` short-circuits the pipeline.
