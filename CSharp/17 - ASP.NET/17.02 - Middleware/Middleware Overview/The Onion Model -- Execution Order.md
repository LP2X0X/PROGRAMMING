---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
---

## The Onion Model -- Execution Order

The way middleware executes is often described as the **"onion model"** or **"Russian doll" model**. Each middleware wraps around the next one, like layers of an onion. Code before `await next()` runs on the way in (request phase), and code after `await next()` runs on the way out (response phase).

```
                         The Onion Model
  
             +---------------------------------------+
             |          Middleware 1 (outer)          |
             |   +-------------------------------+   |
             |   |      Middleware 2 (middle)     |   |
             |   |   +-----------------------+   |   |
             |   |   |   Middleware 3 (inner) |   |   |
             |   |   |                       |   |   |
  Request ===+===+===+===>  Endpoint    ====>+===+===+===> Response
             |   |   |                       |   |   |
             |   |   +-----------------------+   |   |
             |   +-------------------------------+   |
             +---------------------------------------+

  Execution Order:
    IN:   Middleware 1 -> Middleware 2 -> Middleware 3 -> Endpoint
    OUT:  Middleware 3 -> Middleware 2 -> Middleware 1 -> Client
```

Here is a concrete example demonstrating the execution order:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.Use(async (context, next) =>
{
    Console.WriteLine("1. Middleware A - BEFORE next()");
    await next();
    Console.WriteLine("6. Middleware A - AFTER next()");
});

app.Use(async (context, next) =>
{
    Console.WriteLine("2. Middleware B - BEFORE next()");
    await next();
    Console.WriteLine("5. Middleware B - AFTER next()");
});

app.Use(async (context, next) =>
{
    Console.WriteLine("3. Middleware C - BEFORE next()");
    await next();
    Console.WriteLine("4. Middleware C - AFTER next()");
});

app.Run(async context =>
{
    Console.WriteLine("   >>> Endpoint reached <<<");
    await context.Response.WriteAsync("Hello from the endpoint");
});

app.Run();
```

**Console output for a single request:**

```
1. Middleware A - BEFORE next()
2. Middleware B - BEFORE next()
3. Middleware C - BEFORE next()
   >>> Endpoint reached <<<
4. Middleware C - AFTER next()
5. Middleware B - AFTER next()
6. Middleware A - AFTER next()
```

Notice how the "AFTER" messages print in **reverse order**. This is the onion model in action -- the outermost layer is first to see the request and last to see the response.

> [!warning] Common Misconception
> Many beginners think middleware runs top-to-bottom and then stops. In reality, each middleware runs **twice** -- once on the way in (before `next()`) and once on the way out (after `next()`). If you place response-modifying code before `next()`, it may be overwritten by a later middleware. If you place request-reading code after `next()`, the request may have already been consumed.

> [!summary] Section Summary
> Middleware follows the onion model: each component wraps around the next. Code before `await next()` executes during the request phase (outside-in), and code after `await next()` executes during the response phase (inside-out, reverse order). This two-phase execution is critical to understand for correct middleware behavior.
