---
tags: [csharp, asp-net-core, environments, configuration]
---


ASP.NET Core provides extension methods on `IHostEnvironment` (exposed as `app.Environment` or `builder.Environment`) to check the current environment.

### Built-In Check Methods

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    // Only runs in Development
    Console.WriteLine("Running in Development mode");
}

if (app.Environment.IsStaging())
{
    // Only runs in Staging
    Console.WriteLine("Running in Staging mode");
}

if (app.Environment.IsProduction())
{
    // Only runs in Production
    Console.WriteLine("Running in Production mode");
}
```

### Checking for Custom Environments

For custom environment names, use the `IsEnvironment()` method:

```csharp
if (app.Environment.IsEnvironment("QA"))
{
    Console.WriteLine("Running in QA mode");
}

if (app.Environment.IsEnvironment("UAT"))
{
    Console.WriteLine("Running in UAT mode");
}
```

> [!ad-note] Case Insensitivity
> All environment name comparisons are **case-insensitive** on Windows and Linux. `"Development"`, `"development"`, and `"DEVELOPMENT"` are all treated as the same environment.

### Reading the Raw Environment Name

```csharp
string currentEnv = app.Environment.EnvironmentName;
Console.WriteLine($"Current environment: {currentEnv}");
```

### Using Environment in Dependency Injection

You can inject `IHostEnvironment` or `IWebHostEnvironment` into any service:

```csharp
public class OrderService
{
    private readonly IWebHostEnvironment _env;

    public OrderService(IWebHostEnvironment env)
    {
        _env = env;
    }

    public void ProcessOrder(Order order)
    {
        if (_env.IsDevelopment())
        {
            // Log verbose debug information
            Console.WriteLine($"Processing order {order.Id} with {order.Items.Count} items");
        }

        // ... production logic
    }
}
```

> [!summary] Section Summary
> - Use `IsDevelopment()`, `IsStaging()`, and `IsProduction()` for the three built-in environments.
> - Use `IsEnvironment("CustomName")` for custom environments.
> - Comparisons are case-insensitive.
> - Inject `IWebHostEnvironment` to check the environment from any service class.
