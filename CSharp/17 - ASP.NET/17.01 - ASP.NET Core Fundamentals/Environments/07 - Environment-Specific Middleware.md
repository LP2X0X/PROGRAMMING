---
tags: [csharp, asp-net-core, environments, configuration]
---


One of the most important uses of environments is configuring different middleware pipelines for development versus production.

### The Developer Exception Page

In Development, you want detailed error pages that show stack traces, query strings, headers, and routing information. In Production, you want a generic error page that reveals nothing.

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    // Shows detailed exception info: stack trace, source code, query params, cookies, headers
    app.UseDeveloperExceptionPage();
}
else
{
    // Shows a user-friendly error page with no technical details
    app.UseExceptionHandler("/Error");

    // Adds Strict-Transport-Security header for production HTTPS
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

### Other Environment-Conditional Middleware

```csharp
if (app.Environment.IsDevelopment())
{
    // Swagger UI for API documentation during development
    app.UseSwagger();
    app.UseSwaggerUI(options =>
    {
        options.SwaggerEndpoint("/swagger/v1/swagger.json", "Order API v1");
    });

    // Database error page for EF Core migration issues
    app.UseMigrationsEndPoint();
}

if (!app.Environment.IsDevelopment())
{
    // Response compression is usually handled by a reverse proxy in production,
    // but you might enable it here for staging/production
    app.UseResponseCompression();
}
```

> [!example] Typical Middleware Pipeline by Environment
> **Development**: UseDeveloperExceptionPage, UseSwagger, UseSwaggerUI, UseMigrationsEndPoint
> **Staging**: UseExceptionHandler, UseHsts, UseResponseCompression
> **Production**: UseExceptionHandler, UseHsts, UseResponseCompression

> [!summary] Section Summary
> - `UseDeveloperExceptionPage()` provides rich error details and should only run in Development.
> - `UseExceptionHandler("/Error")` provides a safe, generic error page for Staging and Production.
> - Swagger, migration endpoints, and other development tools should be gated behind `IsDevelopment()`.
> - `UseHsts()` sends the HSTS header and is appropriate for non-Development environments.
