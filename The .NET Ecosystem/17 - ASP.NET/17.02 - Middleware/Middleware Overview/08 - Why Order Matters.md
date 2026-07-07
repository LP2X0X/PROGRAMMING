---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
---

## Why Order Matters

The order in which you register middleware in `Program.cs` is the order in which it executes. **Getting the order wrong is one of the most common sources of bugs in ASP.NET Core applications.**

### The Recommended Order

Microsoft recommends this ordering for typical applications:

```csharp
var builder = WebApplication.CreateBuilder(args);
// ... service registration ...
var app = builder.Build();

// 1. Exception/Error handling (outermost -- catches everything)
app.UseExceptionHandler("/error");

// 2. HSTS (HTTP Strict Transport Security)
app.UseHsts();

// 3. HTTPS Redirection
app.UseHttpsRedirection();

// 4. Static Files (serve CSS, JS, images before hitting routing)
app.UseStaticFiles();

// 5. Routing (matches the request to an endpoint)
app.UseRouting();

// 6. CORS (must be after routing, before auth)
app.UseCors();

// 7. Authentication (who are you?)
app.UseAuthentication();

// 8. Authorization (are you allowed?)
app.UseAuthorization();

// 9. Custom middleware
app.Use(async (context, next) => { /* ... */ await next(); });

// 10. Endpoint execution
app.MapControllers();

app.Run();
```

### Concrete Examples of Bugs from Wrong Ordering

**Bug 1: Authorization before Authentication**

```csharp
// WRONG: Authorization runs before the user is identified
app.UseAuthorization();   // Checks permissions -- but User is null!
app.UseAuthentication();  // Identifies the user -- too late

// What happens: Every request to a protected endpoint returns 401 or 403,
// even with valid credentials, because Authorization middleware sees
// an unauthenticated user (context.User has no claims).
```

```csharp
// CORRECT: Authentication first, then Authorization
app.UseAuthentication();  // Identifies the user, populates context.User
app.UseAuthorization();   // Now can check the user's permissions
```

**Bug 2: Static Files after Routing**

```csharp
// WRONG: Static files served after routing
app.UseRouting();
app.UseStaticFiles();     // CSS/JS requests hit the routing pipeline first
app.MapControllers();

// What happens: Requests for /css/site.css go through routing,
// get no match, and return 404 -- even though the file exists.
// Performance also suffers because every static file request
// goes through routing unnecessarily.
```

```csharp
// CORRECT: Static files before routing
app.UseStaticFiles();     // Serves files directly, short-circuits
app.UseRouting();
app.MapControllers();
```

**Bug 3: CORS after Authorization**

```csharp
// WRONG: CORS runs after Authorization
app.UseAuthentication();
app.UseAuthorization();
app.UseCors("AllowFrontend");  // Too late for preflight requests

// What happens: The browser sends an OPTIONS preflight request.
// Authorization middleware rejects it (no auth token on preflight).
// The browser never gets the CORS headers and blocks the real request.
// Your frontend shows: "Access to XMLHttpRequest has been blocked by CORS policy"
```

```csharp
// CORRECT: CORS before Authorization
app.UseAuthentication();
app.UseCors("AllowFrontend");  // Handles preflight before auth checks
app.UseAuthorization();
```

**Bug 4: Exception Handler in the Wrong Position**

```csharp
// WRONG: Exception handler registered too late
app.UseAuthentication();
app.UseAuthorization();
app.UseExceptionHandler("/error");  // Only catches exceptions from middleware below

// What happens: If authentication middleware throws an exception,
// the exception handler never sees it because it is downstream.
// The client receives a raw 500 error with no friendly error page.
```

```csharp
// CORRECT: Exception handler is the outermost middleware
app.UseExceptionHandler("/error");  // Wraps everything -- catches all exceptions
app.UseAuthentication();
app.UseAuthorization();
```

> [!warning] Common Misconception
> Some developers think middleware order only affects performance. In reality, wrong ordering causes **functional bugs** -- authentication failures, CORS errors, missing error pages, and security vulnerabilities. The order is not a suggestion; it is a requirement for correct behavior.

> [!tip]
> When debugging pipeline issues, add a temporary logging middleware at different positions to see what `context.User`, `context.Response.StatusCode`, and `context.Request.Path` look like at each stage. This quickly reveals where things go wrong.

> [!summary] Section Summary
> Middleware order is critical for correct application behavior. Authentication must precede authorization. Static files should come before routing. CORS must handle preflight before authorization rejects it. Exception handling should be the outermost layer to catch all errors. Incorrect ordering causes functional bugs, not just performance issues.
