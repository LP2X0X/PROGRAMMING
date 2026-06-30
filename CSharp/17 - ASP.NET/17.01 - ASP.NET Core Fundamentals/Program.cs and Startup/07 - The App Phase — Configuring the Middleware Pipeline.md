---
tags: [csharp, asp-net-core, startup, program]
---


After calling `builder.Build()`, you receive a `WebApplication` instance. This is where you define the **middleware pipeline** — the sequence of components that handle each HTTP request.

### builder.Build()

```csharp
var app = builder.Build();
```

> [!warning] The Point of No Return
> `builder.Build()` compiles the DI container into an `IServiceProvider`. After this call:
> - You **cannot** register new services.
> - You **cannot** add new configuration sources.
> - You **can** resolve services and configure the middleware pipeline.

### Middleware Order Matters

Middleware components execute in the order they are added. This is one of the most critical concepts in ASP.NET Core.

```csharp
var app = builder.Build();

// 1. Exception handling — must be first to catch all downstream errors
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

// 2. HTTPS redirection
app.UseHttpsRedirection();

// 3. Serve static files (short-circuits for files in wwwroot)
app.UseStaticFiles();

// 4. Routing — matches the request to an endpoint
app.UseRouting();

// 5. CORS — must be between UseRouting and UseAuthorization
app.UseCors("AllowFrontend");

// 6. Authentication — identifies the user
app.UseAuthentication();

// 7. Authorization — checks permissions
app.UseAuthorization();

// 8. Endpoints — executes the matched endpoint
app.MapControllers();
```

> [!warning] Middleware Ordering Rules
> These ordering constraints are enforced by convention, not by the compiler:
> - `UseExceptionHandler` / `UseDeveloperExceptionPage` should be **first**.
> - `UseStaticFiles` should be **before** `UseRouting` (so static files skip auth).
> - `UseCors` must be **after** `UseRouting` and **before** `UseAuthorization`.
> - `UseAuthentication` must come **before** `UseAuthorization`.
> - Endpoint mapping (`MapControllers`, `MapRazorPages`) should be **last**.

### Custom Middleware

You can insert custom middleware anywhere in the pipeline using `app.Use`:

```csharp
// Inline middleware
app.Use(async (context, next) =>
{
    var stopwatch = Stopwatch.StartNew();
    await next(context);
    stopwatch.Stop();

    context.Response.Headers.Append("X-Response-Time", $"{stopwatch.ElapsedMilliseconds}ms");
});

// Class-based middleware (registered via extension method)
app.UseRequestLogging(); // custom extension method
```

> [!example] Request Flow Through the Pipeline
> Consider a request to `GET /api/orders/42`:
> 1. `UseExceptionHandler` wraps the pipeline in a try-catch.
> 2. `UseHttpsRedirection` checks if the request is HTTPS (passes through).
> 3. `UseStaticFiles` checks wwwroot for `/api/orders/42` (not found, passes through).
> 4. `UseRouting` matches the URL to `OrdersController.GetById(42)`.
> 5. `UseCors` applies CORS headers if the origin matches.
> 6. `UseAuthentication` reads the JWT token and sets `HttpContext.User`.
> 7. `UseAuthorization` checks if the user has permission.
> 8. The controller action executes and produces a response.
> 9. The response flows back **up** through each middleware in reverse order.

> [!summary] Section Summary
> - `builder.Build()` freezes the DI container and returns a `WebApplication`.
> - Middleware order is critical: exception handling first, endpoints last.
> - Each `Use*` call adds a middleware component to the pipeline.
> - Requests flow **down** through middleware, responses flow **back up**.
> - Custom middleware can be inline (`app.Use`) or class-based.
