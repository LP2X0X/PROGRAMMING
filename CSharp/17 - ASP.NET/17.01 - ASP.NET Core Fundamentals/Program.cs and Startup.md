---
tags: [csharp, asp-net-core, startup, program]
---

# Program.cs and Startup

> [!ad-note] Overview
> This note covers the **entry point** of every ASP.NET Core application: `Program.cs` (and historically `Startup.cs`). Understanding these files is essential because they control how the application is built, configured, and run. Every service registration, middleware pipeline decision, and hosting configuration flows through here.

## Table of Contents
- [[#Historical Context — The Two-File Model]]
- [[#The Modern Minimal Hosting Model (.NET 6+)]]
- [[#WebApplication.CreateBuilder Explained]]
- [[#The Builder Phase — Configuring Services]]
- [[#The App Phase — Configuring the Middleware Pipeline]]
- [[#Mapping Endpoints]]
- [[#app.Run — Starting the Application]]
- [[#Side-by-Side Comparison — Old vs New]]
- [[#Top-Level Statements Explained]]
- [[#Real-World Program.cs for an MVC Application]]
- [[#Common Pitfalls and Debugging]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]

---

## Historical Context — The Two-File Model

Before .NET 6, ASP.NET Core applications used **two separate files** to configure the application:

1. **Program.cs** — Created and configured the host (Kestrel, logging, configuration sources).
2. **Startup.cs** — Registered services (`ConfigureServices`) and built the middleware pipeline (`Configure`).

### Pre-.NET 6 Program.cs

```csharp
// Program.cs (.NET 5 and earlier)
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

### Pre-.NET 6 Startup.cs

```csharp
// Startup.cs (.NET 5 and earlier)
public class Startup
{
    public IConfiguration Configuration { get; }

    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllersWithViews();
        services.AddDbContext<InventoryContext>(options =>
            options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
        services.AddScoped<IOrderService, OrderService>();
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }
        else
        {
            app.UseExceptionHandler("/Home/Error");
            app.UseHsts();
        }

        app.UseHttpsRedirection();
        app.UseStaticFiles();
        app.UseRouting();
        app.UseAuthentication();
        app.UseAuthorization();

        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllerRoute(
                name: "default",
                pattern: "{controller=Home}/{action=Index}/{id?}");
        });
    }
}
```

> [!ad-note] Why Two Files?
> The two-file model enforced separation of concerns: `Program.cs` dealt with **hosting** (how the app runs) while `Startup.cs` dealt with **application logic** (what services exist and how requests flow). This was a clean design, but it introduced ceremony and indirection that made simple apps harder to set up.

> [!summary] Section Summary
> - Pre-.NET 6 apps used `Program.cs` (host creation) and `Startup.cs` (services + pipeline).
> - `Startup.ConfigureServices` registered DI services; `Startup.Configure` built the middleware pipeline.
> - The `IHostBuilder` pattern drove host creation through `Host.CreateDefaultBuilder`.
> - This model was clean but verbose for small applications.

---

## The Modern Minimal Hosting Model (.NET 6+)

Starting with .NET 6, the ASP.NET Core team introduced the **minimal hosting model**. This collapses `Program.cs` and `Startup.cs` into a single `Program.cs` file using **top-level statements**.

```csharp
// Program.cs (.NET 6+ minimal hosting model)
var builder = WebApplication.CreateBuilder(args);

// === Builder Phase: Configure Services ===
builder.Services.AddControllersWithViews();
builder.Services.AddDbContext<InventoryContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
builder.Services.AddScoped<IOrderService, OrderService>();

var app = builder.Build();

// === App Phase: Configure Middleware Pipeline ===
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

> [!tip] Key Insight
> The minimal hosting model is not a different framework. It is the **same ASP.NET Core** with a simplified bootstrapping API. Under the hood, `WebApplication.CreateBuilder` still creates an `IHostBuilder` and calls `Host.CreateDefaultBuilder`. The simplification is entirely at the API surface.

> [!summary] Section Summary
> - .NET 6+ merges both files into a single `Program.cs` using top-level statements.
> - The same two-phase pattern exists: builder phase (services) then app phase (pipeline).
> - The underlying infrastructure (`IHostBuilder`, `IHost`) is unchanged.
> - Less ceremony, same power.

---

## WebApplication.CreateBuilder Explained

`WebApplication.CreateBuilder(args)` is the starting point for every modern ASP.NET Core application. It returns a `WebApplicationBuilder` that pre-configures a large number of defaults.

### What CreateBuilder Does Automatically

| Component | Default Configuration |
|---|---|
| **Configuration** | Loads `appsettings.json`, `appsettings.{Environment}.json`, user secrets (dev), environment variables, command-line args |
| **Logging** | Console, Debug, EventSource, EventLog (Windows) |
| **DI Container** | Creates the built-in `IServiceCollection` / `IServiceProvider` |
| **Web Server** | Configures Kestrel as the default web server |
| **Content Root** | Sets to the current directory |
| **Environment** | Reads `ASPNETCORE_ENVIRONMENT` (defaults to `Production`) |
| **Host** | Uses `GenericHost` infrastructure internally |

```csharp
var builder = WebApplication.CreateBuilder(args);

// At this point, all of the above are already configured.
// You can now customize any of them:

// Override Kestrel settings
builder.WebHost.ConfigureKestrel(options =>
{
    options.ListenAnyIP(5000);
    options.ListenAnyIP(5001, listenOptions =>
    {
        listenOptions.UseHttps();
    });
});

// Add additional configuration sources
builder.Configuration.AddJsonFile("custom-settings.json", optional: true);

// Change the content root
builder.Environment.ContentRootPath = "/app/data";
```

> [!warning] Builder vs. App
> You can only modify `builder.Services`, `builder.Configuration`, `builder.Logging`, and `builder.WebHost` **before** calling `builder.Build()`. Once `Build()` is called, the DI container is frozen and attempting to register new services will throw an exception.

### WebApplicationOptions Overload

You can also pass `WebApplicationOptions` for finer control:

```csharp
var builder = WebApplication.CreateBuilder(new WebApplicationOptions
{
    Args = args,
    ApplicationName = "InventoryManagement",
    ContentRootPath = Directory.GetCurrentDirectory(),
    EnvironmentName = Environments.Staging,
    WebRootPath = "wwwroot"
});
```

> [!summary] Section Summary
> - `CreateBuilder` sets up configuration, logging, DI, Kestrel, and host infrastructure automatically.
> - Configuration loads from multiple sources in a layered order (later sources override earlier ones).
> - `WebApplicationOptions` allows overriding defaults like environment name and content root.
> - All builder customizations must happen **before** `Build()`.

---

## The Builder Phase — Configuring Services

The builder phase is where you register all services with the [[17.03 - Dependency Injection|dependency injection container]]. This corresponds to the old `Startup.ConfigureServices` method.

### Registering Application Services

```csharp
var builder = WebApplication.CreateBuilder(args);

// Framework services
builder.Services.AddControllersWithViews();
builder.Services.AddRazorPages();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Database context
builder.Services.AddDbContext<InventoryContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("InventoryDb")));

// Application services with different lifetimes
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddScoped<IPaymentProcessor, StripePaymentProcessor>();
builder.Services.AddSingleton<ICacheService, RedisCacheService>();
builder.Services.AddTransient<IEmailSender, SmtpEmailSender>();

// Authentication and authorization
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidateAudience = true,
            ValidAudience = builder.Configuration["Jwt:Audience"],
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]!))
        };
    });
builder.Services.AddAuthorization();

// CORS
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", policy =>
    {
        policy.WithOrigins("https://inventory.example.com")
              .AllowAnyHeader()
              .AllowAnyMethod();
    });
});
```

### Accessing Configuration During Service Registration

`builder.Configuration` gives you access to the `IConfiguration` instance during the builder phase. This is how you read connection strings, API keys, and other settings.

```csharp
// Bind a configuration section to a strongly-typed options class
builder.Services.Configure<SmtpSettings>(
    builder.Configuration.GetSection("SmtpSettings"));

// Read a single value
string apiKey = builder.Configuration["ExternalApi:Key"]
    ?? throw new InvalidOperationException("API key not configured.");
```

> [!ad-note] Configuration Layering
> Configuration sources are loaded in order. Later sources override earlier ones. The typical order is:
> 1. `appsettings.json`
> 2. `appsettings.{Environment}.json`
> 3. User secrets (Development only)
> 4. Environment variables
> 5. Command-line arguments
>
> This means a command-line argument always wins over a value in `appsettings.json`.

### Configuring Logging

```csharp
// Clear default providers and add specific ones
builder.Logging.ClearProviders();
builder.Logging.AddConsole();
builder.Logging.AddDebug();
builder.Logging.SetMinimumLevel(LogLevel.Information);

// Or use a third-party provider like Serilog
builder.Host.UseSerilog((context, configuration) =>
{
    configuration
        .ReadFrom.Configuration(context.Configuration)
        .WriteTo.Console()
        .WriteTo.File("logs/inventory-.log", rollingInterval: RollingInterval.Day);
});
```

> [!summary] Section Summary
> - The builder phase registers all services into the DI container via `builder.Services`.
> - Use `AddScoped`, `AddSingleton`, `AddTransient` for application services.
> - Framework services like MVC, EF Core, and authentication each have `Add*` extension methods.
> - `builder.Configuration` provides access to all configuration sources during registration.
> - `builder.Logging` and `builder.Host` allow customizing logging and host-level concerns.

---

## The App Phase — Configuring the Middleware Pipeline

After calling `builder.Build()`, you receive a `WebApplication` instance. This is where you define the **middleware pipeline** — the sequence of components that handle each HTTP request.

### builder.Build()

```csharp
var app = builder.Build();
```

> [!warning] The Point of No Return
> `builder.Build()` compiles the DI container into an `IServiceProvider`. After this call:
> - You **cannot** register new services.
> - You **cannot** add new configuration sources.
> - You **can** resolve services and configure the middleware pipeline.

### Middleware Order Matters

Middleware components execute in the order they are added. This is one of the most critical concepts in ASP.NET Core.

```csharp
var app = builder.Build();

// 1. Exception handling — must be first to catch all downstream errors
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

// 3. Serve static files (short-circuits for files in wwwroot)
app.UseStaticFiles();

// 4. Routing — matches the request to an endpoint
app.UseRouting();

// 5. CORS — must be between UseRouting and UseAuthorization
app.UseCors("AllowFrontend");

// 6. Authentication — identifies the user
app.UseAuthentication();

// 7. Authorization — checks permissions
app.UseAuthorization();

// 8. Endpoints — executes the matched endpoint
app.MapControllers();
```

> [!warning] Middleware Ordering Rules
> These ordering constraints are enforced by convention, not by the compiler:
> - `UseExceptionHandler` / `UseDeveloperExceptionPage` should be **first**.
> - `UseStaticFiles` should be **before** `UseRouting` (so static files skip auth).
> - `UseCors` must be **after** `UseRouting` and **before** `UseAuthorization`.
> - `UseAuthentication` must come **before** `UseAuthorization`.
> - Endpoint mapping (`MapControllers`, `MapRazorPages`) should be **last**.

### Custom Middleware

You can insert custom middleware anywhere in the pipeline using `app.Use`:

```csharp
// Inline middleware
app.Use(async (context, next) =>
{
    var stopwatch = Stopwatch.StartNew();
    await next(context);
    stopwatch.Stop();

    context.Response.Headers.Append("X-Response-Time", $"{stopwatch.ElapsedMilliseconds}ms");
});

// Class-based middleware (registered via extension method)
app.UseRequestLogging(); // custom extension method
```

> [!example] Request Flow Through the Pipeline
> Consider a request to `GET /api/orders/42`:
> 1. `UseExceptionHandler` wraps the pipeline in a try-catch.
> 2. `UseHttpsRedirection` checks if the request is HTTPS (passes through).
> 3. `UseStaticFiles` checks wwwroot for `/api/orders/42` (not found, passes through).
> 4. `UseRouting` matches the URL to `OrdersController.GetById(42)`.
> 5. `UseCors` applies CORS headers if the origin matches.
> 6. `UseAuthentication` reads the JWT token and sets `HttpContext.User`.
> 7. `UseAuthorization` checks if the user has permission.
> 8. The controller action executes and produces a response.
> 9. The response flows back **up** through each middleware in reverse order.

> [!summary] Section Summary
> - `builder.Build()` freezes the DI container and returns a `WebApplication`.
> - Middleware order is critical: exception handling first, endpoints last.
> - Each `Use*` call adds a middleware component to the pipeline.
> - Requests flow **down** through middleware, responses flow **back up**.
> - Custom middleware can be inline (`app.Use`) or class-based.

---

## Mapping Endpoints

The `app.Map*()` methods define which code handles which URLs. This replaces the old `app.UseEndpoints(endpoints => { ... })` pattern.

### MVC Controllers

```csharp
// Map all controller routes using attribute routing
app.MapControllers();

// Map conventional MVC routes
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

// Map Razor Pages
app.MapRazorPages();
```

### Minimal API Endpoints

```csharp
// Simple GET endpoint
app.MapGet("/api/health", () => Results.Ok(new { Status = "Healthy" }));

// GET with route parameter and DI
app.MapGet("/api/orders/{id:int}", async (int id, IOrderService orderService) =>
{
    var order = await orderService.GetByIdAsync(id);
    return order is not null
        ? Results.Ok(order)
        : Results.NotFound();
});

// POST with request body
app.MapPost("/api/orders", async (CreateOrderRequest request, IOrderService orderService) =>
{
    var orderId = await orderService.CreateAsync(request);
    return Results.Created($"/api/orders/{orderId}", new { Id = orderId });
});

// Grouping endpoints
var orderGroup = app.MapGroup("/api/orders")
    .RequireAuthorization();

orderGroup.MapGet("/", async (IOrderService svc) => await svc.GetAllAsync());
orderGroup.MapGet("/{id:int}", async (int id, IOrderService svc) => await svc.GetByIdAsync(id));
orderGroup.MapPost("/", async (CreateOrderRequest req, IOrderService svc) => await svc.CreateAsync(req));
```

> [!ad-note] MapGroup
> `MapGroup` (introduced in .NET 7) lets you define a common prefix and apply shared filters (like authorization) to a group of endpoints. This keeps your `Program.cs` organized as the number of minimal API endpoints grows.

> [!summary] Section Summary
> - `MapControllers` / `MapControllerRoute` wire up MVC controllers.
> - `MapGet`, `MapPost`, `MapPut`, `MapDelete` define minimal API endpoints.
> - `MapGroup` organizes related endpoints under a common prefix with shared configuration.
> - Minimal APIs support parameter binding from route, query, body, and DI automatically.

---

## app.Run — Starting the Application

The final call in `Program.cs` is `app.Run()`, which starts the web server and blocks the calling thread until shutdown.

```csharp
app.Run();
```

### What app.Run() Does

1. Starts Kestrel (or the configured web server).
2. Begins listening on the configured URLs and ports.
3. Blocks the main thread, keeping the application alive.
4. Listens for shutdown signals (`Ctrl+C`, `SIGTERM`, `app.StopAsync()`).
5. Triggers graceful shutdown: stops accepting new connections, drains existing requests, disposes services.

### Specifying URLs

```csharp
// Via app.Run parameter
app.Run("https://localhost:7001");

// Via builder configuration
builder.WebHost.UseUrls("https://localhost:5001", "http://localhost:5000");

// Via environment variable
// ASPNETCORE_URLS=https://localhost:5001;http://localhost:5000

// Via command-line argument
// dotnet run --urls "https://localhost:5001"
```

### RunAsync for Non-Blocking Scenarios

```csharp
// Useful in integration tests or background service scenarios
await app.RunAsync();
```

> [!tip] Run vs RunAsync vs Start
> - `app.Run()` blocks the calling thread until the app shuts down.
> - `app.RunAsync()` returns a `Task` that completes on shutdown (useful with `await`).
> - `app.Start()` / `app.StartAsync()` starts the server without blocking — you manage the lifetime yourself.

> [!summary] Section Summary
> - `app.Run()` starts Kestrel and blocks until the application shuts down.
> - URLs can be configured via code, environment variables, or command-line arguments.
> - Graceful shutdown drains existing requests before terminating.
> - `RunAsync` and `StartAsync` offer non-blocking alternatives for advanced scenarios.

---

## Side-by-Side Comparison — Old vs New

| Aspect | Pre-.NET 6 (Startup.cs) | .NET 6+ (Minimal Hosting) |
|---|---|---|
| **Files** | `Program.cs` + `Startup.cs` | Single `Program.cs` |
| **Entry point** | `Main` method with `CreateHostBuilder` | Top-level statements |
| **Service registration** | `Startup.ConfigureServices(IServiceCollection)` | `builder.Services.Add*()` |
| **Pipeline config** | `Startup.Configure(IApplicationBuilder, IWebHostEnvironment)` | `app.Use*()` directly |
| **Host builder** | `Host.CreateDefaultBuilder(args)` | `WebApplication.CreateBuilder(args)` |
| **Endpoint mapping** | `app.UseEndpoints(e => e.MapControllers())` | `app.MapControllers()` directly |
| **Environment check** | Inject `IWebHostEnvironment env` parameter | `app.Environment.IsDevelopment()` |
| **Configuration access** | Inject `IConfiguration` in constructor | `builder.Configuration` directly |

> [!ad-note] Can You Still Use Startup.cs in .NET 6+?
> Yes. The minimal hosting model is the **default**, but you can still use the `Startup` class pattern if you prefer. However, the .NET templates no longer scaffold it, and the community has largely adopted minimal hosting.

> [!summary] Section Summary
> - The two-file model and single-file model are functionally equivalent.
> - The minimal model reduces boilerplate without sacrificing capability.
> - Service registration and pipeline configuration follow the same patterns in both.
> - The `Startup.cs` class pattern is still supported but no longer the default.

---

## Top-Level Statements Explained

The minimal hosting model relies on C# 9's **top-level statements** feature. This allows a `.cs` file to contain executable code without wrapping it in a class or `Main` method.

### How It Works

```csharp
// This is a complete, valid C# program:
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();
app.MapGet("/", () => "Hello, World!");
app.Run();
```

The compiler generates an implicit `Program` class with a `Main` method behind the scenes.

> [!ad-note] The Hidden Program Class
> Even with top-level statements, a `Program` class exists at compile time. This matters for:
> - Integration testing: you reference `Program` as the entry point assembly.
> - Accessing the class in test projects: add `InternalsVisibleTo` or make `Program` partial.

```csharp
// At the bottom of Program.cs, add this for integration test access:
public partial class Program { }
```

### args Is Available

The `args` parameter (command-line arguments) is implicitly available in top-level statements. You do not need to declare it.

```csharp
// args is available without declaration
var builder = WebApplication.CreateBuilder(args);
Console.WriteLine($"Started with {args.Length} arguments");
```

> [!summary] Section Summary
> - Top-level statements eliminate the `class Program` and `static void Main` boilerplate.
> - The compiler generates an implicit `Program` class with a `Main` method.
> - `args` is implicitly available for command-line argument access.
> - Add `public partial class Program { }` for integration test compatibility.

---

## Real-World Program.cs for an MVC Application

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

---

## Common Pitfalls and Debugging

### Registering Services After Build

```csharp
var app = builder.Build();
// This will throw an InvalidOperationException:
builder.Services.AddScoped<IOrderService, OrderService>();
```

> [!warning] Fix
> Always register services **before** `builder.Build()`. If you see "Cannot modify ServiceCollection after application is built," move the registration above the `Build()` call.

### Wrong Middleware Order

```csharp
// BUG: Authorization runs before Authentication
app.UseAuthorization();
app.UseAuthentication(); // Too late -- user identity not yet established
```

> [!warning] Fix
> `UseAuthentication()` must always come **before** `UseAuthorization()`. The auth middleware sets `HttpContext.User`, which the authorization middleware then inspects.

### Missing await on RunAsync

```csharp
// BUG: Application exits immediately
app.RunAsync(); // Not awaited -- Main returns, process exits
```

> [!warning] Fix
> Either use `app.Run()` (blocking) or `await app.RunAsync()` (non-blocking but awaited). Never call `RunAsync()` without `await` unless you are managing the lifetime yourself.

### Static Files Not Serving

```csharp
// BUG: Static files middleware is after authorization
app.UseAuthentication();
app.UseAuthorization();
app.UseStaticFiles(); // CSS/JS now require authentication
```

> [!warning] Fix
> Place `UseStaticFiles()` **before** `UseAuthentication()` and `UseAuthorization()` so that static assets in `wwwroot` are served without requiring login.

> [!summary] Section Summary
> - Never register services after calling `Build()`.
> - Middleware order bugs are silent at compile time but break behavior at runtime.
> - Always `await` `RunAsync()` or use the blocking `Run()`.
> - Place `UseStaticFiles()` early in the pipeline to avoid requiring authentication for assets.

---

## Comprehensive Summary

> [!tip] Complete Summary
> **Program.cs** is the single entry point for modern ASP.NET Core applications. It replaces the older two-file model (`Program.cs` + `Startup.cs`) with a streamlined approach using top-level statements and the minimal hosting API.
>
> The application lifecycle follows a **two-phase pattern**:
> 1. **Builder Phase** — `WebApplication.CreateBuilder(args)` sets up configuration, logging, DI, and Kestrel. You register all services via `builder.Services`, access configuration via `builder.Configuration`, and customize logging via `builder.Logging`.
> 2. **App Phase** — `builder.Build()` freezes the DI container and returns a `WebApplication`. You then configure the middleware pipeline with `app.Use*()` calls (order matters critically) and map endpoints with `app.Map*()` calls.
>
> `app.Run()` starts Kestrel and blocks until the application shuts down gracefully.
>
> Key rules to remember:
> - All service registration happens **before** `Build()`.
> - Middleware order determines request handling behavior: exception handling first, endpoints last, authentication before authorization.
> - `CreateBuilder` provides sensible defaults (Kestrel, layered configuration, console logging) that cover most scenarios.
> - The old `Startup.cs` pattern is still supported but the community has standardized on the minimal hosting model.
> - For integration tests, add `public partial class Program { }` at the bottom of `Program.cs`.

---

## Related Topics

- [[ASP.NET Core Overview]] — High-level architecture of ASP.NET Core
- [[Project Structure]] — How files and folders are organized in ASP.NET Core projects
- [[Hosting Model]] — Kestrel, IIS integration, and reverse proxy patterns
- [[Environments]] — Development, Staging, Production and environment-based configuration
- [[17.03 - Dependency Injection]] — Service lifetimes, registration patterns, and the DI container
- [[Middleware]] — How the middleware pipeline processes requests
- [[Configuration]] — appsettings.json, user secrets, environment variables, and the Options pattern
- [[Logging]] — Built-in logging, Serilog, and structured logging
- [[Minimal APIs]] — Lightweight endpoint mapping without controllers
