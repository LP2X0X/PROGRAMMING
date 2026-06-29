---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - fundamentals
---

## The builder.Services Property

In modern ASP.NET Core (using the minimal hosting model introduced in .NET 6), all service registration happens through `builder.Services` in `Program.cs`.

```csharp
var builder = WebApplication.CreateBuilder(args);

// builder.Services is an IServiceCollection
// All registrations go here, before builder.Build()

// Framework services
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Application services
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddScoped<ICustomerService, CustomerService>();
builder.Services.AddScoped<IInventoryRepository, SqlInventoryRepository>();
builder.Services.AddSingleton<IPaymentGateway, StripePaymentGateway>();

// Third-party and infrastructure services
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// After Build(), you configure the middleware pipeline
app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

> [!tip] Organizing Registrations
> As your application grows, `Program.cs` can become cluttered with service registrations. A common pattern is to use **extension methods** to group related registrations:
>
> ```csharp
> // In a separate file: ServiceCollectionExtensions.cs
> public static class ServiceCollectionExtensions
> {
>     public static IServiceCollection AddOrderingServices(this IServiceCollection services)
>     {
>         services.AddScoped<IOrderService, OrderService>();
>         services.AddScoped<IInventoryRepository, SqlInventoryRepository>();
>         services.AddScoped<IPaymentGateway, StripePaymentGateway>();
>         services.AddTransient<IEmailService, SmtpEmailService>();
>         return services;
>     }
>
>     public static IServiceCollection AddCustomerServices(this IServiceCollection services)
>     {
>         services.AddScoped<ICustomerService, CustomerService>();
>         services.AddScoped<ICustomerRepository, SqlCustomerRepository>();
>         return services;
>     }
> }
>
> // In Program.cs -- clean and readable
> builder.Services.AddOrderingServices();
> builder.Services.AddCustomerServices();
> ```

> [!warning] Common Misconception
> You might wonder if the order of registrations matters. For most cases, it does not -- the container resolves dependencies based on types, not registration order. However, if you register the **same interface twice**, the last registration wins (the container returns the last-registered implementation). If you need multiple implementations of the same interface, you can inject `IEnumerable<IMyService>` to get all of them.

> [!summary] Section Summary
> - `builder.Services` is the `IServiceCollection` where all service registrations happen in `Program.cs`.
> - Both framework services (`AddControllers`, `AddDbContext`) and your application services are registered here.
> - All registrations must happen before `builder.Build()` is called.
> - Use extension methods on `IServiceCollection` to keep `Program.cs` clean as your application grows.
> - If the same interface is registered multiple times, the last registration wins for single-instance resolution.
