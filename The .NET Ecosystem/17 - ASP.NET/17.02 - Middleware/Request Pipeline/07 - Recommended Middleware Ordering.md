---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
  - request
---

## Recommended Middleware Ordering

Microsoft defines a standard ordering for middleware. This is not arbitrary -- each position exists for a specific technical reason. Here is the canonical order:

```csharp
var builder = WebApplication.CreateBuilder(args);

// ... service registration ...

var app = builder.Build();

// 1. Exception Handling (outermost wrapper)
app.UseExceptionHandler("/error");

// 2. HSTS (Strict Transport Security header)
app.UseHsts();

// 3. HTTPS Redirection
app.UseHttpsRedirection();

// 4. Static Files
app.UseStaticFiles();

// 5. Routing (selects endpoint)
app.UseRouting();

// 6. CORS
app.UseCors();

// 7. Authentication (identifies the user)
app.UseAuthentication();

// 8. Authorization (checks permissions)
app.UseAuthorization();

// 9. Custom middleware (your own logic)
app.UseMiddleware<RequestLoggingMiddleware>();

// 10. Endpoints (executes the selected endpoint)
app.MapControllers();
app.MapRazorPages();
```

> [!summary] Section Summary
> The ten-position standard ordering ensures that errors are caught at the outermost layer, security redirects happen before content is served, static files bypass unnecessary processing, routing selects endpoints before authorization checks them, and custom logic runs just before endpoint execution.
