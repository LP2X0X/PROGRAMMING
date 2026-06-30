---
tags: [csharp, asp-net-core, startup, program]
---


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
