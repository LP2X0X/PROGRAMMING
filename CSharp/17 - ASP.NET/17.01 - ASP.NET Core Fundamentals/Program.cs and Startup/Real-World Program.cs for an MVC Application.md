---
tags: [csharp, asp-net-core, startup, program]
---


Below is a production-realistic `Program.cs` for an inventory management MVC application with authentication, Entity Framework Core, logging, health checks, and more.

```csharp
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using Serilog;
using InventoryManagement.Data;
using InventoryManagement.Services;

// --- Configure Serilog early for startup logging ---
Log.Logger = new LoggerConfiguration()
    .WriteTo.Console()
    .CreateBootstrapLogger();

try
{
    Log.Information("Starting InventoryManagement application");

    var builder = WebApplication.CreateBuilder(args);

    // === Logging ===
    builder.Host.UseSerilog((context, services, configuration) =>
    {
        configuration
            .ReadFrom.Configuration(context.Configuration)
            .ReadFrom.Services(services)
            .Enrich.FromLogContext()
            .WriteTo.Console()
            .WriteTo.File("logs/inventory-.log", rollingInterval: RollingInterval.Day);
    });

    // === Database ===
    builder.Services.AddDbContext<InventoryContext>(options =>
        options.UseSqlServer(
            builder.Configuration.GetConnectionString("InventoryDb"),
            sqlOptions => sqlOptions.EnableRetryOnFailure(maxRetryCount: 3)));

    // === Identity / Authentication ===
    builder.Services.AddDefaultIdentity<IdentityUser>(options =>
    {
        options.Password.RequireDigit = true;
        options.Password.RequiredLength = 8;
        options.Lockout.MaxFailedAccessAttempts = 5;
        options.SignIn.RequireConfirmedEmail = true;
    })
    .AddRoles<IdentityRole>()
    .AddEntityFrameworkStores<InventoryContext>();

    // === Application Services ===
    builder.Services.AddScoped<IOrderService, OrderService>();
    builder.Services.AddScoped<IInventoryTracker, InventoryTracker>();
    builder.Services.AddScoped<INotificationService, EmailNotificationService>();
    builder.Services.AddSingleton<IProductCacheService, ProductCacheService>();

    // === Options Pattern ===
    builder.Services.Configure<SmtpSettings>(
        builder.Configuration.GetSection("SmtpSettings"));
    builder.Services.Configure<InventoryThresholds>(
        builder.Configuration.GetSection("InventoryThresholds"));

    // === MVC + Razor ===
    builder.Services.AddControllersWithViews();
    builder.Services.AddRazorPages();

    // === Health Checks ===
    builder.Services.AddHealthChecks()
        .AddDbContextCheck<InventoryContext>("database")
        .AddCheck<ExternalApiHealthCheck>("external-api");

    // === CORS ===
    builder.Services.AddCors(options =>
    {
        options.AddPolicy("Dashboard", policy =>
            policy.WithOrigins(
                    builder.Configuration.GetSection("Cors:AllowedOrigins").Get<string[]>()
                    ?? Array.Empty<string>())
                .AllowAnyHeader()
                .AllowAnyMethod()
                .AllowCredentials());
    });

    // ==============================
    // BUILD
    // ==============================
    var app = builder.Build();

    // === Middleware Pipeline ===
    if (app.Environment.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
        app.UseMigrationsEndPoint();
    }
    else
    {
        app.UseExceptionHandler("/Home/Error");
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseSerilogRequestLogging();

    app.UseRouting();
    app.UseCors("Dashboard");
    app.UseAuthentication();
    app.UseAuthorization();

    // === Endpoints ===
    app.MapControllerRoute(
        name: "areas",
        pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}");
    app.MapControllerRoute(
        name: "default",
        pattern: "{controller=Home}/{action=Index}/{id?}");
    app.MapRazorPages();
    app.MapHealthChecks("/health");

    app.Run();
}
catch (Exception ex)
{
    Log.Fatal(ex, "Application terminated unexpectedly");
}
finally
{
    Log.CloseAndFlush();
}
```

> [!tip] Structured Startup Logging
> Wrapping the entire `Program.cs` in a try-catch with Serilog ensures that startup failures (database connection errors, missing configuration, port conflicts) are **logged** rather than silently lost in the console.

> [!summary] Section Summary
> - A real-world `Program.cs` follows the builder-then-app pattern with many service registrations.
> - Structured logging (Serilog) should be configured early, even before the builder.
> - Health checks, CORS, Identity, EF Core, and custom services all register in the builder phase.
> - Wrapping startup in try-catch ensures fatal errors are captured.
