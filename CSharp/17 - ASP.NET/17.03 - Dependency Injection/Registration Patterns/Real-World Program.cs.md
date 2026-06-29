---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - patterns
---

## Real-World Program.cs

Here is a realistic `Program.cs` service registration section that combines the patterns covered above, organized with extension methods and logical grouping.

```csharp
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// -------------------------------------------------------
// Framework services
// -------------------------------------------------------
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// -------------------------------------------------------
// Database
// -------------------------------------------------------
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")));

// Generic repository (open generics)
builder.Services.AddScoped(typeof(IRepository<>), typeof(SqlRepository<>));

// -------------------------------------------------------
// Application services (grouped by feature)
// -------------------------------------------------------
builder.Services.AddOrderProcessing();
builder.Services.AddInventoryManagement();
builder.Services.AddNotifications(options =>
{
    options.DefaultChannel = "email";
    options.RetryCount = 3;
});
builder.Services.AddPaymentProcessing();

// -------------------------------------------------------
// Configuration (Options pattern)
// -------------------------------------------------------
builder.Services.AddOptions<SmtpSettings>()
    .BindConfiguration("Smtp")
    .ValidateDataAnnotations()
    .ValidateOnStart();

builder.Services.AddOptions<PaymentGatewaySettings>()
    .BindConfiguration("PaymentGateway")
    .ValidateDataAnnotations()
    .ValidateOnStart();

// -------------------------------------------------------
// Keyed services (.NET 8+)
// -------------------------------------------------------
builder.Services.AddKeyedScoped<IPaymentProcessor, StripePaymentProcessor>("stripe");
builder.Services.AddKeyedScoped<IPaymentProcessor, PayPalPaymentProcessor>("paypal");
builder.Services.AddKeyedScoped<IPaymentProcessor, SquarePaymentProcessor>("square");

// -------------------------------------------------------
// Infrastructure
// -------------------------------------------------------
builder.Services.AddSingleton<ICacheService, RedisCacheService>();
builder.Services.AddHttpClient<IExternalPricingApi, ExternalPricingApiClient>(client =>
{
    client.BaseAddress = new Uri(
        builder.Configuration["ExternalApis:Pricing:BaseUrl"]!);
    client.Timeout = TimeSpan.FromSeconds(10);
});

// -------------------------------------------------------
// Cross-cutting (decorators)
// -------------------------------------------------------
builder.Services.Decorate<IOrderService, LoggingOrderService>();

// -------------------------------------------------------
// Build and configure pipeline
// -------------------------------------------------------
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### The extension methods behind the scenes

```csharp
public static class OrderProcessingServiceExtensions
{
    public static IServiceCollection AddOrderProcessing(
        this IServiceCollection services)
    {
        services.AddScoped<IOrderService, OrderService>();
        services.AddScoped<IOrderValidator, OrderValidator>();
        services.AddScoped<IOrderPricingEngine, OrderPricingEngine>();
        services.AddTransient<IOrderConfirmationSender, EmailOrderConfirmationSender>();

        return services;
    }
}

public static class InventoryServiceExtensions
{
    public static IServiceCollection AddInventoryManagement(
        this IServiceCollection services)
    {
        services.AddScoped<IInventoryService, InventoryService>();
        services.AddScoped<IWarehouseLocator, WarehouseLocator>();
        services.AddScoped<IStockLevelChecker, StockLevelChecker>();

        return services;
    }
}

public static class PaymentServiceExtensions
{
    public static IServiceCollection AddPaymentProcessing(
        this IServiceCollection services)
    {
        services.AddScoped<IPaymentService, PaymentService>();
        services.AddScoped<IRefundProcessor, RefundProcessor>();
        services.AddScoped<IPaymentAuditLogger, PaymentAuditLogger>();

        return services;
    }
}
```

> [!ad-note]
> Notice the structure: framework services at the top, then database, then application features (via extension methods), then configuration, then infrastructure, then cross-cutting concerns. This ordering is a convention, not a requirement, but it makes `Program.cs` easy to navigate. Each extension method file lives alongside the feature code it registers, not in a central "registration" folder.

> [!summary] Section Summary
> - A well-organized `Program.cs` groups registrations by category with comment separators.
> - Feature-specific registrations are delegated to `Add{Feature}()` extension methods.
> - Framework services, database, options, keyed services, infrastructure, and cross-cutting concerns each get their own section.
> - Extension method classes live alongside the feature code they register.
> - This structure scales well as the application grows -- new features add new extension methods without bloating `Program.cs`.
