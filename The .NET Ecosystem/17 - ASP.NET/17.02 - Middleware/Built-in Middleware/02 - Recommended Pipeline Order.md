---
tags:
  - csharp
  - asp-net-core
  - middleware
  - built-in
---

## Recommended Pipeline Order

The order in which you register middleware determines the order they execute. ASP.NET Core does not enforce order -- it is your responsibility. The following is the **recommended order** from the official documentation:

```csharp
var app = builder.Build();

// 1. Exception handling (outermost layer)
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

// 3. Static files (short-circuit before routing)
app.UseStaticFiles();

// 4. Routing (matches URL to endpoint)
app.UseRouting();

// 5. CORS (must be between routing and authorization)
app.UseCors();

// 6. Authentication and Authorization
app.UseAuthentication();
app.UseAuthorization();

// 7. Session
app.UseSession();

// 8. Response caching / compression
app.UseResponseCaching();

// 9. Rate limiting
app.UseRateLimiter();

// 10. Endpoint execution (controllers, minimal APIs, Razor Pages)
app.MapControllers();
app.MapRazorPages();
```

> [!warning]
> Placing middleware out of order is one of the most common sources of subtle bugs in ASP.NET Core applications. For example, placing `UseAuthorization` before `UseRouting` will cause authorization to fail because no endpoint has been matched yet.

> [!summary] Section Summary
> The recommended pipeline order ensures that each middleware has the context it needs from earlier middleware. Exception handling wraps everything, HTTPS and static files come before routing, and authentication must precede authorization.
