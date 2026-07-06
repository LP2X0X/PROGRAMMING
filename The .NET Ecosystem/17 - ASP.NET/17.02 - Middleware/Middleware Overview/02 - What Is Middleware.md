---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
---

## What Is Middleware

A **middleware component** is a piece of code that sits in the HTTP request pipeline and has access to both the incoming `HttpRequest` and the outgoing `HttpResponse`. Each middleware component can:

1. **Inspect** the request (read headers, check authentication tokens, log information)
2. **Modify** the request or response (add headers, transform the body, set status codes)
3. **Pass the request** to the next middleware in the pipeline by calling `next()`
4. **Short-circuit** the pipeline by NOT calling `next()`, which means no further middleware runs

Think of middleware as a series of checkpoints at an airport. Each checkpoint examines you, possibly stamps your passport, and either lets you proceed to the next checkpoint or turns you away. On your way back out, you pass through the same checkpoints in reverse.

```csharp
// A simple middleware that logs request timing
app.Use(async (context, next) =>
{
    var stopwatch = System.Diagnostics.Stopwatch.StartNew();
    
    // Code here runs BEFORE the next middleware (on the way IN)
    Console.WriteLine($"[Request] {context.Request.Method} {context.Request.Path}");
    
    await next();  // Pass control to the next middleware
    
    // Code here runs AFTER the next middleware (on the way OUT)
    stopwatch.Stop();
    Console.WriteLine($"[Response] {context.Response.StatusCode} in {stopwatch.ElapsedMilliseconds}ms");
});
```

> [!info] Definition
> **Middleware** = a component assembled into an application pipeline to handle requests and responses. Each component chooses whether to pass the request to the next component and can perform work before and after the next component in the pipeline.

> [!summary] Section Summary
> Middleware components are the building blocks of the ASP.NET Core request pipeline. Each component can inspect, modify, pass along, or short-circuit HTTP requests and responses. They execute in a defined order, giving you full control over how requests are processed.
