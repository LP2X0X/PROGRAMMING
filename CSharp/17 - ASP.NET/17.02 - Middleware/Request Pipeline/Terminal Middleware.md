---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
  - request
---

## Terminal Middleware

**Terminal middleware** is middleware registered with `app.Run()`. It NEVER calls `next()` -- it always ends the pipeline.

```csharp
// Terminal middleware: always responds, never passes to next
app.Run(async context =>
{
    context.Response.ContentType = "text/plain";
    await context.Response.WriteAsync("This is a terminal response. Nothing else will run.");
});
```

### Key Characteristics of `app.Run()`

1. It does not receive a `next` parameter -- there is no option to call the next middleware
2. Any middleware registered AFTER `app.Run()` is never reached
3. It is typically used as a catch-all fallback at the end of the pipeline

### Practical Use: Fallback for Unmatched Requests

```csharp
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
app.MapRazorPages();

// Terminal fallback: if nothing above handled the request
app.Run(async context =>
{
    context.Response.StatusCode = StatusCodes.Status404NotFound;
    context.Response.ContentType = "application/json";
    await context.Response.WriteAsJsonAsync(new
    {
        Error = "Resource not found",
        Path = context.Request.Path.Value,
        Timestamp = DateTime.UtcNow
    });
});
```

> [!warning] Common Misconception
> `app.Run()` is NOT the same as `app.Use()`. With `app.Use()`, you receive a `next` delegate and CHOOSE whether to call it. With `app.Run()`, there is no `next` parameter at all. It is structurally impossible for terminal middleware to pass the request along.

> [!summary] Section Summary
> Terminal middleware (`app.Run()`) always ends the pipeline. It does not receive a `next` delegate. Use it as a catch-all fallback for requests that no other middleware or endpoint handled.
