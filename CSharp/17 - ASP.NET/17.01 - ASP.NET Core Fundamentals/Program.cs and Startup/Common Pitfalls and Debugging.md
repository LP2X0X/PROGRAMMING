---
tags: [csharp, asp-net-core, startup, program]
---


### Registering Services After Build

```csharp
var app = builder.Build();
// This will throw an InvalidOperationException:
builder.Services.AddScoped<IOrderService, OrderService>();
```

> [!warning] Fix
> Always register services **before** `builder.Build()`. If you see "Cannot modify ServiceCollection after application is built," move the registration above the `Build()` call.

### Wrong Middleware Order

```csharp
// BUG: Authorization runs before Authentication
app.UseAuthorization();
app.UseAuthentication(); // Too late -- user identity not yet established
```

> [!warning] Fix
> `UseAuthentication()` must always come **before** `UseAuthorization()`. The auth middleware sets `HttpContext.User`, which the authorization middleware then inspects.

### Missing await on RunAsync

```csharp
// BUG: Application exits immediately
app.RunAsync(); // Not awaited -- Main returns, process exits
```

> [!warning] Fix
> Either use `app.Run()` (blocking) or `await app.RunAsync()` (non-blocking but awaited). Never call `RunAsync()` without `await` unless you are managing the lifetime yourself.

### Static Files Not Serving

```csharp
// BUG: Static files middleware is after authorization
app.UseAuthentication();
app.UseAuthorization();
app.UseStaticFiles(); // CSS/JS now require authentication
```

> [!warning] Fix
> Place `UseStaticFiles()` **before** `UseAuthentication()` and `UseAuthorization()` so that static assets in `wwwroot` are served without requiring login.

> [!summary] Section Summary
> - Never register services after calling `Build()`.
> - Middleware order bugs are silent at compile time but break behavior at runtime.
> - Always `await` `RunAsync()` or use the blocking `Run()`.
> - Place `UseStaticFiles()` early in the pipeline to avoid requiring authentication for assets.
