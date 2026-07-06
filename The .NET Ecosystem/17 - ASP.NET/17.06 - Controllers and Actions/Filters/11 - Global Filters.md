---
tags:
  - csharp
  - asp-net-core
  - filters
  - controllers
  - cross-cutting
---


Global filters apply to **every controller and every action** without needing attributes. Register them in `Program.cs`:

```csharp
builder.Services.AddControllers(options =>
{
    // Add filter by type (resolved from DI if registered, otherwise activated)
    options.Filters.Add<LogActionFilter>();
    options.Filters.Add<ApiExceptionFilter>();

    // Add filter instance directly (must be parameterless or pre-constructed)
    options.Filters.Add(new ResponseHeaderFilter());

    // Add with explicit order
    options.Filters.Add<PerformanceTimingFilter>(order: 1);
});
```

```ad-note
Global filters are useful for concerns that truly apply everywhere: exception handling, performance logging, security headers, etc. For concerns that apply to a subset of actions, prefer attribute-based application.
```
