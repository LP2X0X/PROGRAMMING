---
tags:
  - csharp
  - asp-net-core
  - middleware
  - built-in
---

## UseDeveloperExceptionPage

**`UseDeveloperExceptionPage`** renders a detailed, interactive error page when an unhandled exception occurs. It shows the full stack trace, source code snippets around the exception, request headers, cookies, query strings, and routing data.

### Configuration

```csharp
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
```

> [!danger]
> **Never enable `UseDeveloperExceptionPage` in production.** It exposes internal implementation details including:
> - Full stack traces with file paths and line numbers
> - Source code snippets
> - Environment variable names
> - Connection strings visible in exception messages
> - Internal class and method names
>
> This information is invaluable for attackers performing reconnaissance on your application.

### What the Page Shows

- **Exception details**: type, message, and full stack trace
- **Source code**: the exact lines around where the exception was thrown (if PDB files are available)
- **Query string**: all query parameters
- **Cookies**: all cookies sent with the request
- **Headers**: all HTTP request headers
- **Routing**: matched route data

### Automatic in .NET 6+

In .NET 6 and later, the developer exception page is automatically enabled when the application runs in the `Development` environment. You do not need to explicitly call `UseDeveloperExceptionPage()` if you are using `WebApplication.CreateBuilder()` -- it is added by default. However, explicit registration makes the intent clear.

### When You Need It

Only during local development and debugging.

### Gotchas

- If both `UseDeveloperExceptionPage` and `UseExceptionHandler` are registered, the one registered first wins because it catches the exception before the other sees it
- The developer exception page **does not log exceptions** -- it only displays them. You still need logging middleware or `ILogger` in production

> [!summary] Section Summary
> `UseDeveloperExceptionPage` is a powerful debugging aid that shows comprehensive error details. It is strictly for development use and is automatically enabled in .NET 6+ development environments. Never deploy it to production.
