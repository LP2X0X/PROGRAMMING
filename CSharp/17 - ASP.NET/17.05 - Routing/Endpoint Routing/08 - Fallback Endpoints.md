---
tags:
  - csharp
  - asp-net-core
  - routing
  - endpoints
---


Fallback endpoints match when **no other route** matches the request. They are essential for Single Page Applications (SPAs) and custom 404 handling.

### SPA Fallback

```csharp
app.UseStaticFiles();      // Serve static files first
app.UseRouting();

app.MapControllers();
app.MapFallbackToFile("index.html");  // Everything else -> SPA entry point
```

When a request does not match any API route or static file, `MapFallbackToFile` serves the SPA's `index.html`, allowing the client-side router to handle the URL.

### Custom Fallback Logic

```csharp
app.MapFallback(async context =>
{
    context.Response.StatusCode = 404;
    await context.Response.WriteAsJsonAsync(new
    {
        error = "Not Found",
        message = $"No endpoint matches '{context.Request.Path}'"
    });
});
```

### Fallback to a Controller Action

```csharp
app.MapFallbackToController("Index", "Spa");
// Routes unmatched requests to SpaController.Index()
```

### Fallback to a Razor Page

```csharp
app.MapFallbackToPage("/Error");
// Routes unmatched requests to Pages/Error.cshtml
```

> [!warning] Common Misconception
> Fallback routes have the **lowest possible priority**. They only match after every other route has been checked. However, `UseStaticFiles()` runs **before** `UseRouting()`, so static files (CSS, JS, images) are served directly without hitting the fallback. If you place `UseStaticFiles()` after `UseRouting()`, static file requests may hit the fallback instead.

> [!summary] Section Summary
> - `MapFallbackToFile("index.html")` serves the SPA entry point for unmatched routes.
> - `MapFallback()` provides custom fallback logic.
> - Fallback endpoints have the lowest priority -- they only match when nothing else does.
> - `UseStaticFiles()` must come before `UseRouting()` to prevent static files from hitting the fallback.
