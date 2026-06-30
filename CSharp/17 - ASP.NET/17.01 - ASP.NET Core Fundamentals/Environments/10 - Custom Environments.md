---
tags: [csharp, asp-net-core, environments, configuration]
---


You are not limited to the three built-in environment names. You can create any custom environment by setting `ASPNETCORE_ENVIRONMENT` to a custom string.

### Common Custom Environments

| Environment | Use Case                                                        |
|---|---|
| QA          | Quality assurance testing with test data                        |
| UAT         | User acceptance testing with near-production configuration      |
| Local       | Alternative to Development for distinguishing CI from local     |
| CI          | Continuous integration pipelines                                |
| Performance | Load testing and performance benchmarking                       |

### Setting Up a Custom Environment

1. Create the configuration file `appsettings.QA.json`:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "ConnectionStrings": {
    "InventoryDb": "Server=qa-db-server;Database=InventoryDb_QA;Trusted_Connection=True;"
  },
  "FeatureFlags": {
    "EnableNewCheckout": true,
    "EnableBetaDashboard": true
  }
}
```

2. Check for it in code:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else if (app.Environment.IsEnvironment("QA"))
{
    // QA gets the generic error handler but with additional diagnostics
    app.UseExceptionHandler("/Error");
    app.UseSwagger();
    app.UseSwaggerUI();
}
else
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}
```

3. Set the environment variable when deploying to the QA server:

```bash
export ASPNETCORE_ENVIRONMENT=QA
dotnet OrderService.dll
```

> [!ad-note] No Built-In Helper for Custom Environments
> There is no `IsQA()` or `IsUAT()` method. You must use `IsEnvironment("QA")` for custom environment names. You could create your own extension methods if you use custom environments frequently:
> ```csharp
> public static class HostEnvironmentExtensions
> {
>     public static bool IsQA(this IHostEnvironment env)
>         => env.IsEnvironment("QA");
>
>     public static bool IsUAT(this IHostEnvironment env)
>         => env.IsEnvironment("UAT");
> }
> ```

> [!summary] Section Summary
> - You can define any custom environment name beyond the three defaults.
> - Create a matching `appsettings.{EnvironmentName}.json` for custom configuration.
> - Use `IsEnvironment("Name")` to check for custom environments in code.
> - Consider creating extension methods for frequently used custom environments.
