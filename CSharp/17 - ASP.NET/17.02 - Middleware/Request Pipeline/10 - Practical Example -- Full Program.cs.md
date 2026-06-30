---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
  - request
---

## Practical Example -- Full Program.cs

Here is a complete, production-style `Program.cs` with all middleware in the correct order, annotated with detailed comments:

```csharp
using Microsoft.AspNetCore.HttpOverrides;

var builder = WebApplication.CreateBuilder(args);

// ============================================================
// SERVICE REGISTRATION
// ============================================================
builder.Services.AddControllers();
builder.Services.AddRazorPages();
builder.Services.AddSignalR();

builder.Services.AddAuthentication("Bearer")
    .AddJwtBearer(options =>
    {
        options.Authority = "https://auth.example.com";
        options.Audience = "order-api";
    });

builder.Services.AddAuthorizationBuilder()
    .AddPolicy("OrderViewer", policy => policy.RequireClaim("permission", "orders.read"))
    .AddPolicy("OrderManager", policy => policy.RequireClaim("permission", "orders.write"))
    .AddPolicy("AdminOnly", policy => policy.RequireRole("Admin"));

builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowOrderDashboard", policy =>
    {
        policy.WithOrigins("https://dashboard.example.com")
              .AllowAnyHeader()
              .AllowAnyMethod()
              .AllowCredentials();
    });
});

builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddScoped<IInventoryService, InventoryService>();

var app = builder.Build();

// ============================================================
// MIDDLEWARE PIPELINE (ORDER MATTERS!)
// ============================================================

// --------------------------------------------------
// 1. EXCEPTION HANDLING (outermost -- catches everything)
// --------------------------------------------------
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/error");
}

// --------------------------------------------------
// 2. HSTS (tell browsers to use HTTPS only)
// --------------------------------------------------
if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
}

// --------------------------------------------------
// 3. HTTPS REDIRECTION (force secure connections)
// --------------------------------------------------
app.UseHttpsRedirection();

// --------------------------------------------------
// 4. FORWARDED HEADERS (if behind a reverse proxy)
// --------------------------------------------------
app.UseForwardedHeaders(new ForwardedHeadersOptions
{
    ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto
});

// --------------------------------------------------
// 5. STATIC FILES (short-circuits for wwwroot files)
// --------------------------------------------------
app.UseStaticFiles();

// --------------------------------------------------
// 6. ROUTING (selects the endpoint -- does NOT execute it)
// --------------------------------------------------
app.UseRouting();

// --------------------------------------------------
// 7. CORS (must be after routing, before auth)
// --------------------------------------------------
app.UseCors("AllowOrderDashboard");

// --------------------------------------------------
// 8. AUTHENTICATION (identifies the user)
// --------------------------------------------------
app.UseAuthentication();

// --------------------------------------------------
// 9. AUTHORIZATION (checks permissions for selected endpoint)
// --------------------------------------------------
app.UseAuthorization();

// --------------------------------------------------
// 10. CUSTOM MIDDLEWARE (runs in authenticated context)
// --------------------------------------------------
app.UseMiddleware<RequestLoggingMiddleware>();
app.UseMiddleware<TenantResolutionMiddleware>();

// --------------------------------------------------
// 11. ENDPOINTS (executes the selected endpoint)
// --------------------------------------------------
app.MapControllers();
app.MapRazorPages();
app.MapHub<OrderNotificationHub>("/hubs/orders");

// Health check endpoint (no auth required)
app.MapGet("/health", () => Results.Ok(new { Status = "Healthy", Timestamp = DateTime.UtcNow }))
   .AllowAnonymous();

// --------------------------------------------------
// 12. TERMINAL FALLBACK (catches unmatched requests)
// --------------------------------------------------
app.Run(async context =>
{
    context.Response.StatusCode = StatusCodes.Status404NotFound;
    context.Response.ContentType = "application/json";
    await context.Response.WriteAsJsonAsync(new
    {
        Error = "The requested resource was not found",
        Path = context.Request.Path.Value
    });
});

app.Run();
```

> [!tip] Practical Tip
> In production, you often also need `UseForwardedHeaders()` when running behind a reverse proxy like Nginx or a load balancer. Place it after HTTPS redirection but before static files. Without it, the application may see internal IP addresses and HTTP scheme instead of the client's real IP and HTTPS.

> [!summary] Section Summary
> A production `Program.cs` follows the ten-position ordering strictly. Exception handling wraps everything, security middleware runs early, static files short-circuit, routing selects endpoints, CORS and auth run in the routing split, custom middleware runs in authenticated context, and endpoints execute last.
