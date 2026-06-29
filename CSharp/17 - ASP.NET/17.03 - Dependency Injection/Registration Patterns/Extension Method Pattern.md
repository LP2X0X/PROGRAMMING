---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - patterns
---

## Extension Method Pattern

As your `Program.cs` grows, grouping related service registrations into **extension methods** keeps the file organized and readable. This is the standard pattern used by ASP.NET Core itself (e.g., `AddControllers()`, `AddDbContext()`, `AddAuthentication()`).

### Creating a registration extension method

```csharp
public static class OrderProcessingServiceExtensions
{
    public static IServiceCollection AddOrderProcessing(
        this IServiceCollection services)
    {
        services.AddScoped<IOrderService, OrderService>();
        services.AddScoped<IOrderRepository, SqlOrderRepository>();
        services.AddScoped<IOrderValidator, OrderValidator>();
        services.AddScoped<IInventoryService, InventoryService>();
        services.AddTransient<IOrderConfirmationSender, EmailOrderConfirmationSender>();

        return services;
    }
}
```

### Using the extension method in Program.cs

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddOrderProcessing();
builder.Services.AddNotifications();
builder.Services.AddPaymentProcessing();
builder.Services.AddInventoryManagement();

var app = builder.Build();
```

### Extension method with configuration options

```csharp
public static class NotificationServiceExtensions
{
    public static IServiceCollection AddNotifications(
        this IServiceCollection services,
        Action<NotificationOptions>? configure = null)
    {
        if (configure is not null)
        {
            services.Configure(configure);
        }

        services.AddScoped<INotifier, EmailNotifier>();
        services.AddScoped<INotifier, SmsNotifier>();
        services.AddScoped<NotificationDispatcher>();
        services.AddSingleton<INotificationTemplateEngine, RazorNotificationTemplateEngine>();

        return services;
    }
}

// Usage
builder.Services.AddNotifications(options =>
{
    options.DefaultChannel = "email";
    options.RetryCount = 3;
});
```

> [!tip]
> Follow these conventions for registration extension methods:
> - Name the method `Add{Feature}` to match the ASP.NET Core convention.
> - Place it in a `static class` named `{Feature}ServiceExtensions` or `{Feature}ServiceCollectionExtensions`.
> - Always return `IServiceCollection` to enable method chaining.
> - Put the extension class in the `Microsoft.Extensions.DependencyInjection` namespace (for library code) or your application namespace (for app code).

> [!summary] Section Summary
> - Group related registrations into `static` extension methods on `IServiceCollection`.
> - Name them `Add{Feature}()` to follow the ASP.NET Core convention.
> - Always return `IServiceCollection` for method chaining.
> - Optionally accept an `Action<TOptions>` parameter for configurable registration.
> - This pattern keeps `Program.cs` clean and makes registration modules reusable.
