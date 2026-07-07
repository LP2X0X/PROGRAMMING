---
tags: [csharp, asp-net-core, fundamentals, web]
---


ASP.NET Core processes every HTTP request through a **middleware pipeline**. Each middleware component can:

1. Handle the request and short-circuit the pipeline
2. Pass the request to the next middleware in the chain
3. Execute logic before AND after the next middleware (wrapping pattern)

```
Request --> Middleware 1 --> Middleware 2 --> Middleware 3 --> Endpoint
              |                |                |
              v                v                v
Response <-- Middleware 1 <-- Middleware 2 <-- Middleware 3
```

> [!warning] Order Matters
> Middleware order is critical. For example, `UseAuthentication()` must come before `UseAuthorization()`, and both must come before `MapControllers()`. Incorrect ordering leads to subtle bugs -- such as authorization running before the user's identity is established.

### Common Middleware Order

```csharp
var app = builder.Build();

// 1. Exception handling (outermost -- catches everything)
app.UseExceptionHandler("/error");

// 2. HSTS (HTTP Strict Transport Security)
app.UseHsts();

// 3. HTTPS redirection
app.UseHttpsRedirection();

// 4. Static files (short-circuits for CSS, JS, images)
app.UseStaticFiles();

// 5. Routing (matches the request to an endpoint)
app.UseRouting();

// 6. CORS (must be between routing and auth)
app.UseCors();

// 7. Authentication (who are you?)
app.UseAuthentication();

// 8. Authorization (are you allowed?)
app.UseAuthorization();

// 9. Endpoint execution
app.MapControllers();
app.MapRazorPages();
```

> [!tip] Custom Middleware
> You can write your own middleware for cross-cutting concerns like request logging, correlation IDs, or tenant resolution:
> ```csharp
> app.Use(async (context, next) =>
> {
>     var correlationId = Guid.NewGuid().ToString();
>     context.Response.Headers["X-Correlation-Id"] = correlationId;
>
>     // Before the next middleware
>     var stopwatch = Stopwatch.StartNew();
>
>     await next(context);
>
>     // After the next middleware
>     stopwatch.Stop();
>     Console.WriteLine(
>         $"Request {context.Request.Path} took {stopwatch.ElapsedMilliseconds}ms");
> });
> ```

> [!summary] Section Summary
> - Every request flows through a composable middleware pipeline
> - Each middleware can handle, pass through, or wrap the request
> - Middleware order is critical -- authentication before authorization before endpoints
> - Custom middleware is straightforward to write for cross-cutting concerns
> - Static files middleware short-circuits the pipeline for non-dynamic content
