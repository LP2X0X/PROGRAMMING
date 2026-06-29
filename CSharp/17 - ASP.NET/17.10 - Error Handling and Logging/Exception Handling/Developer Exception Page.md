---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - exceptions
  - middleware
---


The **Developer Exception Page** is a built-in middleware that provides rich, detailed error information during development. It is enabled by default in the Development environment in .NET 6+ project templates.

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// In .NET 6+, this is automatically added when
// app.Environment.IsDevelopment() is true.
// You can also add it explicitly:
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
```

When an unhandled exception occurs, this middleware catches it and renders a detailed HTML page that includes:

| Information | Description |
|---|---|
| **Stack trace** | Full call stack with source file paths and line numbers |
| **Source code** | The actual source code lines around where the exception occurred |
| **Request details** | HTTP method, URL, query string, headers |
| **Cookies** | All cookies sent with the request |
| **Route data** | Route values, endpoint metadata |
| **Exception details** | Exception type, message, inner exceptions |

> [!danger]
> The Developer Exception Page must **never** be enabled in production. It exposes source code, file paths, configuration details, and potentially sensitive data. The `IsDevelopment()` check is critical. If your environment variable `ASPNETCORE_ENVIRONMENT` is not set, the default is `Production`, which is the safe default.

> [!ad-note]
> In .NET 6 and later with `WebApplication.CreateBuilder()`, the Developer Exception Page is added automatically when the environment is Development. You do not need to call `app.UseDeveloperExceptionPage()` explicitly unless you are using `WebApplication.CreateSlimBuilder()` or a custom hosting setup.

> [!summary] Section Summary
> - The Developer Exception Page provides rich error details including stack traces, source code, request info, and cookies
> - It is automatically enabled in the Development environment in .NET 6+ templates
> - It must never be active in production due to the sensitive information it exposes
> - The `ASPNETCORE_ENVIRONMENT` variable controls which environment is active
