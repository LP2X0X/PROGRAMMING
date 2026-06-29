---
tags: [csharp, asp-net-core, startup, program]
---


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
