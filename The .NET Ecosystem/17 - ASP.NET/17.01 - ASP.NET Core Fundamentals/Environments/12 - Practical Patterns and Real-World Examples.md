---
tags: [csharp, asp-net-core, environments, configuration]
---


### Complete Program.cs with Environment-Aware Configuration

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register services with environment-aware implementations
builder.Services.AddControllersWithViews();
builder.Services.AddDbContext<InventoryContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("InventoryDb")));

if (builder.Environment.IsDevelopment())
{
    builder.Services.AddSingleton<IEmailSender, ConsoleEmailSender>();
    builder.Services.AddSingleton<IPaymentGateway, SandboxPaymentGateway>();
}
else
{
    builder.Services.AddSingleton<IEmailSender, SmtpEmailSender>();
    builder.Services.AddSingleton<IPaymentGateway, StripePaymentGateway>();
}

var app = builder.Build();

// Configure the HTTP request pipeline
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
    app.UseSwagger();
    app.UseSwaggerUI();
    app.UseMigrationsEndPoint();
}
else if (app.Environment.IsStaging())
{
    app.UseExceptionHandler("/Error");
    // Staging may still want Swagger for QA testers
    app.UseSwagger();
    app.UseSwaggerUI();
}
else
{
    app.UseExceptionHandler("/Error");
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

### Using IOptions Pattern with Environment-Specific Settings

```csharp
// In appsettings.json
// {
//   "OrderProcessing": {
//     "MaxRetries": 3,
//     "TimeoutSeconds": 30,
//     "EnableNotifications": true
//   }
// }

// In appsettings.Development.json
// {
//   "OrderProcessing": {
//     "MaxRetries": 1,
//     "TimeoutSeconds": 5,
//     "EnableNotifications": false
//   }
// }

public class OrderProcessingSettings
{
    public int MaxRetries { get; set; }
    public int TimeoutSeconds { get; set; }
    public bool EnableNotifications { get; set; }
}

// Registration
builder.Services.Configure<OrderProcessingSettings>(
    builder.Configuration.GetSection("OrderProcessing"));

// Usage in a service
public class OrderProcessor
{
    private readonly OrderProcessingSettings _settings;

    public OrderProcessor(IOptions<OrderProcessingSettings> settings)
    {
        _settings = settings.Value;
    }

    public async Task ProcessAsync(Order order)
    {
        for (int attempt = 0; attempt < _settings.MaxRetries; attempt++)
        {
            // Process with environment-appropriate timeout and retry settings
        }
    }
}
```

### Conditional Database Seeding

```csharp
if (app.Environment.IsDevelopment())
{
    using var scope = app.Services.CreateScope();
    var context = scope.ServiceProvider.GetRequiredService<InventoryContext>();
    context.Database.EnsureCreated();

    if (!context.Products.Any())
    {
        context.Products.AddRange(
            new Product { Name = "Widget A", Price = 9.99m, Stock = 100 },
            new Product { Name = "Widget B", Price = 19.99m, Stock = 50 },
            new Product { Name = "Widget C", Price = 29.99m, Stock = 25 }
        );
        context.SaveChanges();
    }
}
```

> [!summary] Section Summary
> - A complete `Program.cs` ties together environment-specific services, middleware, and configuration.
> - The `IOptions<T>` pattern works naturally with environment-specific JSON files.
> - Database seeding in Development ensures developers always have test data available.
> - Keep environment checks in `Program.cs` rather than scattering them throughout business logic.
