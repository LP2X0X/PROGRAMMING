---
tags: [csharp, asp-net-core, fundamentals, web]
---


### Cross-Platform

ASP.NET Core applications run anywhere .NET runs:

- **Windows** -- IIS, Windows Services, Kestrel standalone
- **Linux** -- systemd services, Docker containers, Kubernetes
- **macOS** -- development and production

```bash
# Create and run an ASP.NET Core app on any OS
dotnet new webapi -n OrderService
cd OrderService
dotnet run
```

### Open Source

The entire ASP.NET Core framework is open source under the MIT license. The source code is on GitHub at `dotnet/aspnetcore`. You can read the source, file issues, and submit pull requests.

### Modular Architecture

ASP.NET Core uses a **middleware pipeline** where you compose your application from small, focused components. Nothing is included by default -- you opt in to what you need.

```csharp
var builder = WebApplication.CreateBuilder(args);

// Only add the services you need
builder.Services.AddControllers();
builder.Services.AddScoped<IOrderRepository, SqlOrderRepository>();
builder.Services.AddDbContext<InventoryContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Inventory")));

var app = builder.Build();

// Only add the middleware you need
app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### High Performance

ASP.NET Core is built on **Kestrel**, a high-performance, cross-platform HTTP server. Key performance features include:

- Kestrel uses asynchronous I/O throughout
- Memory-efficient request processing with `System.IO.Pipelines`
- `Span<T>` and `Memory<T>` used extensively to reduce allocations
- Support for HTTP/2 and HTTP/3
- Native AOT compilation support (from .NET 8) for minimal startup time

> [!tip] TechEmpower Benchmarks
> ASP.NET Core consistently places in the top tier of the TechEmpower web framework benchmarks, often outperforming frameworks from other ecosystems like Node.js Express and Spring Boot in plaintext and JSON serialization tests.

### Built-in Dependency Injection

Unlike ASP.NET Framework where DI was optional and required third-party containers (Unity, Autofac, Ninject), ASP.NET Core has a **first-class DI container** built in. Every service in the framework is registered and resolved through DI.

```csharp
// Registering services
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddSingleton<ICacheService, RedisCacheService>();
builder.Services.AddTransient<IEmailSender, SmtpEmailSender>();
```

See [[17.03 - Dependency Injection|Dependency Injection]] for a deep dive into service lifetimes and registration patterns.

### Unified Configuration System

ASP.NET Core replaces the old `web.config` / `ConfigurationManager` approach with a layered configuration system:

```json
// appsettings.json
{
  "ConnectionStrings": {
    "InventoryDb": "Server=localhost;Database=Inventory;Trusted_Connection=true;"
  },
  "OrderProcessing": {
    "MaxRetryAttempts": 3,
    "TimeoutSeconds": 30
  }
}
```

```csharp
// Reading configuration with the Options pattern
public class OrderProcessingOptions
{
    public int MaxRetryAttempts { get; set; }
    public int TimeoutSeconds { get; set; }
}

// In Program.cs
builder.Services.Configure<OrderProcessingOptions>(
    builder.Configuration.GetSection("OrderProcessing"));

// In a service -- injected via DI
public class OrderService
{
    private readonly OrderProcessingOptions _options;

    public OrderService(IOptions<OrderProcessingOptions> options)
    {
        _options = options.Value;
    }
}
```

> [!summary] Section Summary
> - Cross-platform: runs on Windows, Linux, and macOS
> - Open source under MIT license, hosted on GitHub
> - Modular middleware pipeline -- you compose only what you need
> - High performance via Kestrel, async I/O, Span/Memory, and HTTP/2-3 support
> - Built-in DI container replaces the need for third-party IoC containers
> - Unified configuration system with JSON files, environment variables, and the Options pattern
