---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
  - request
---

## Branching the Pipeline

ASP.NET Core allows you to **branch** the pipeline so that different request paths run through entirely different middleware chains.

### `Map` -- Branch by Path Prefix

`app.Map()` creates a branch that runs only when the request path matches a given prefix:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseExceptionHandler("/error");

// Branch: requests starting with /api get a separate pipeline
app.Map("/api", apiApp =>
{
    apiApp.UseAuthentication();
    apiApp.UseAuthorization();
    apiApp.UseMiddleware<ApiRateLimitingMiddleware>();
    apiApp.UseMiddleware<ApiRequestLoggingMiddleware>();
    apiApp.UseRouting();
    apiApp.MapControllers();
});

// Branch: requests starting with /admin get stricter security
app.Map("/admin", adminApp =>
{
    adminApp.UseAuthentication();
    adminApp.UseAuthorization();
    adminApp.UseMiddleware<AdminAuditMiddleware>();
    adminApp.UseRouting();
    adminApp.MapControllers();
});

// Default pipeline for everything else (public pages)
app.UseStaticFiles();
app.UseRouting();
app.MapRazorPages();

app.Run();
```

> [!ad-note] Path Stripping
> When using `Map("/api", ...)`, the `/api` prefix is **stripped** from the path inside the branch. So a request to `/api/orders` arrives inside the branch as `/orders`. This affects how routing matches controllers and endpoints inside the branch.

### `MapWhen` -- Branch by Condition

`app.MapWhen()` branches based on an arbitrary condition, not just a path prefix:

```csharp
// Branch for requests that accept JSON (API clients)
app.MapWhen(
    context => context.Request.Headers.Accept.ToString().Contains("application/json"),
    jsonApp =>
    {
        jsonApp.UseMiddleware<JsonErrorHandlingMiddleware>();
        jsonApp.UseAuthentication();
        jsonApp.UseAuthorization();
        jsonApp.UseRouting();
        jsonApp.MapControllers();
    }
);

// Branch for requests with a specific tenant header
app.MapWhen(
    context => context.Request.Headers.ContainsKey("X-Tenant-Id"),
    tenantApp =>
    {
        tenantApp.UseMiddleware<TenantResolutionMiddleware>();
        tenantApp.UseAuthentication();
        tenantApp.UseAuthorization();
        tenantApp.UseRouting();
        tenantApp.MapControllers();
    }
);
```

### `UseWhen` -- Conditional Middleware (Rejoins Main Pipeline)

Unlike `Map` and `MapWhen` which create separate pipeline branches, `UseWhen` adds middleware conditionally but **rejoins** the main pipeline afterward:

```csharp
// Add extra logging ONLY for /api requests, then rejoin the main pipeline
app.UseWhen(
    context => context.Request.Path.StartsWithSegments("/api"),
    apiApp =>
    {
        apiApp.UseMiddleware<ApiRequestLoggingMiddleware>();
    }
);

// These still run for ALL requests (including /api)
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
```

| Method | Branch Type | Rejoins Main Pipeline? | Use Case |
|---|---|---|---|
| `Map` | Path prefix | No | Separate pipelines for `/api` vs `/admin` |
| `MapWhen` | Arbitrary condition | No | Different handling based on headers, query strings |
| `UseWhen` | Arbitrary condition | Yes | Add extra middleware conditionally without branching |

> [!summary] Section Summary
> `Map` branches by path prefix (strips the prefix), `MapWhen` branches by arbitrary condition, and both create fully separate pipelines. `UseWhen` adds conditional middleware but rejoins the main pipeline. Use branching when different request types need fundamentally different middleware stacks.
