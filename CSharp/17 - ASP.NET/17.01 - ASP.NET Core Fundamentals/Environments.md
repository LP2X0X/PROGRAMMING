---
tags: [csharp, asp-net-core, environments, configuration]
---

# Environments in ASP.NET Core

> [!ad-note] Overview
> ASP.NET Core uses an **environment** system to control application behavior at runtime. The framework ships with three conventional environments -- Development, Staging, and Production -- but you can define any custom environment you need. The current environment determines which configuration files load, which middleware activates, and how much detail error pages reveal. Understanding environments is essential for building applications that are debuggable in development and secure in production.

## Table of Contents

- [[#The Three Built-In Environments]]
- [[#How the Environment Is Set]]
- [[#launchSettings.json Explained]]
- [[#Checking the Current Environment in Code]]
- [[#Environment-Specific Configuration Files]]
- [[#Environment-Specific Middleware]]
- [[#Environment-Specific Services]]
- [[#Environment Tag Helper in Razor]]
- [[#Why You Must Never Run Development in Production]]
- [[#Custom Environments]]
- [[#Environment Variables vs Command-Line Arguments]]
- [[#Practical Patterns and Real-World Examples]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]

---

## The Three Built-In Environments

ASP.NET Core recognizes three environment names by convention. These are not hard-coded restrictions -- they are simply the names the framework provides helper methods for.

| Environment   | Purpose                                                                 | Typical Behavior                                  |
| ------------- | ----------------------------------------------------------------------- | ------------------------------------------------- |
| Development   | Local developer machines                                                | Detailed errors, hot reload, verbose logging       |
| Staging       | Pre-production testing that mirrors production infrastructure           | Near-production settings with additional diagnostics |
| Production    | Live, customer-facing deployment                                        | Optimized performance, generic errors, minimal logging |

> [!tip] Default Behavior
> If `ASPNETCORE_ENVIRONMENT` is not set at all, ASP.NET Core defaults to **Production**. This is a deliberate safety measure -- if you forget to configure the environment, the application runs in the most restrictive, most secure mode.

```csharp
// The default host builder sets Production when no environment variable is found
var builder = WebApplication.CreateBuilder(args);
// builder.Environment.EnvironmentName == "Production" if nothing else is configured
```

> [!summary] Section Summary
> - ASP.NET Core ships with three conventional environments: Development, Staging, and Production.
> - The default environment is Production when nothing is explicitly set.
> - Each environment drives different configuration, middleware, and error-handling behavior.

---

## How the Environment Is Set

The environment is controlled by the `ASPNETCORE_ENVIRONMENT` environment variable. There are several places you can set it.

### Setting via Environment Variable

On Windows (PowerShell):

```powershell
$env:ASPNETCORE_ENVIRONMENT = "Development"
dotnet run
```

On Windows (Command Prompt):

```bash
set ASPNETCORE_ENVIRONMENT=Development
dotnet run
```

On Linux/macOS:

```bash
export ASPNETCORE_ENVIRONMENT=Development
dotnet run
```

### Setting via launchSettings.json (Development Only)

The most common way during local development is through `launchSettings.json`, which lives in the `Properties` folder of your project.

### Setting via Command-Line Argument

```bash
dotnet run --environment Staging
```

### Setting in Code (Rarely Used)

```csharp
var builder = WebApplication.CreateBuilder(new WebApplicationOptions
{
    EnvironmentName = Environments.Staging
});
```

> [!warning] Precedence Order
> The environment is resolved in this order (last one wins):
> 1. Host configuration defaults (Production)
> 2. `ASPNETCORE_ENVIRONMENT` environment variable
> 3. `launchSettings.json` (only when launched via `dotnet run` from the project directory)
> 4. Command-line arguments (`--environment`)
> 5. Explicit setting in code (`WebApplicationOptions.EnvironmentName`)

### Setting in IIS / Azure / Docker

For IIS, you set the environment variable in `web.config`:

```xml
<aspNetCore processPath="dotnet" arguments=".\OrderService.dll">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Production" />
  </environmentVariables>
</aspNetCore>
```

For Azure App Service, you set it in the Application Settings blade in the Azure portal.

For Docker:

```bash
docker run -e ASPNETCORE_ENVIRONMENT=Production my-order-service:latest
```

> [!summary] Section Summary
> - `ASPNETCORE_ENVIRONMENT` is the primary mechanism for setting the environment.
> - `launchSettings.json` is the standard approach during local development.
> - You can also set it via command-line arguments, code, IIS config, Azure settings, or Docker.
> - When unset, the environment defaults to Production for safety.

---

## launchSettings.json Explained

The `launchSettings.json` file configures how `dotnet run` and Visual Studio launch your application during development. It lives at `Properties/launchSettings.json` and is **not deployed to production**.

### Full Example

```json
{
  "$schema": "http://json.schemastore.org/launchsettings.json",
  "iisSettings": {
    "windowsAuthentication": false,
    "anonymousAuthentication": true,
    "iisExpress": {
      "applicationUrl": "http://localhost:54321",
      "sslPort": 44321
    }
  },
  "profiles": {
    "http": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "applicationUrl": "http://localhost:5100",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "https": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "applicationUrl": "https://localhost:7100;http://localhost:5100",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "IIS Express": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

### Key Properties

| Property               | Description                                                                 |
| ---------------------- | --------------------------------------------------------------------------- |
| `commandName`          | How the app launches: `Project` (Kestrel), `IISExpress`, or `Executable`    |
| `applicationUrl`       | The URL(s) Kestrel listens on; separate multiple with semicolons            |
| `launchBrowser`        | Whether to open the browser automatically on launch                         |
| `environmentVariables` | Dictionary of environment variables set before the app starts               |
| `dotnetRunMessages`    | Whether `dotnet run` displays informational messages in the console         |

> [!ad-note] Profile Selection
> When you run `dotnet run`, the first profile with `"commandName": "Project"` is used by default. You can select a specific profile with `dotnet run --launch-profile "https"`. Visual Studio lets you pick the profile from a dropdown in the toolbar.

> [!warning] Do Not Deploy launchSettings.json
> This file is for local development only. It should not be included in your publish output. The default `.pubxml` and `dotnet publish` already exclude it, but be careful with custom deployment scripts.

> [!summary] Section Summary
> - `launchSettings.json` lives in `Properties/` and configures development-time launch behavior.
> - It defines profiles with application URLs, environment variables, and browser launch settings.
> - It is not deployed to production -- it only affects `dotnet run` and IDE launches.
> - The first `Project` profile is used by default unless you specify `--launch-profile`.

---

## Checking the Current Environment in Code

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

---

## Environment-Specific Configuration Files

ASP.NET Core's configuration system automatically loads environment-specific JSON files that override the base `appsettings.json`.

### Loading Order

The configuration system loads files in this order (later files override earlier ones):

1. `appsettings.json` -- base settings shared across all environments
2. `appsettings.{Environment}.json` -- environment-specific overrides
3. User Secrets (Development only)
4. Environment variables
5. Command-line arguments

### Example: Base Configuration

`appsettings.json`:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "ConnectionStrings": {
    "InventoryDb": "Server=prod-db-server;Database=InventoryDb;Trusted_Connection=True;"
  },
  "EmailSettings": {
    "SmtpServer": "smtp.company.com",
    "Port": 587,
    "EnableSsl": true
  }
}
```

### Example: Development Override

`appsettings.Development.json`:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft.AspNetCore": "Information"
    }
  },
  "ConnectionStrings": {
    "InventoryDb": "Server=localhost;Database=InventoryDb_Dev;Trusted_Connection=True;"
  },
  "EmailSettings": {
    "SmtpServer": "localhost",
    "Port": 1025,
    "EnableSsl": false
  }
}
```

### Example: Staging Override

`appsettings.Staging.json`:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "ConnectionStrings": {
    "InventoryDb": "Server=staging-db-server;Database=InventoryDb_Staging;Trusted_Connection=True;"
  }
}
```

> [!tip] Partial Overrides
> Environment-specific files do not need to repeat every setting from the base file. Only include the keys you want to override. All other values fall through from `appsettings.json`.

> [!warning] Never Store Secrets in appsettings.json
> Connection strings with passwords, API keys, and other secrets should use User Secrets (in Development) or environment variables / Azure Key Vault / secret managers (in Staging and Production). The `appsettings.*.json` files are checked into source control.

### How It Works in Program.cs

The `WebApplication.CreateBuilder(args)` call sets this up automatically:

```csharp
var builder = WebApplication.CreateBuilder(args);

// This is already done for you internally:
// builder.Configuration
//     .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
//     .AddJsonFile($"appsettings.{builder.Environment.EnvironmentName}.json", optional: true, reloadOnChange: true)
//     .AddUserSecrets<Program>(optional: true)  // Development only
//     .AddEnvironmentVariables()
//     .AddCommandLine(args);
```

> [!summary] Section Summary
> - `appsettings.json` provides base configuration for all environments.
> - `appsettings.{Environment}.json` overrides specific keys for that environment.
> - The override is automatic -- no extra code needed in `Program.cs`.
> - Later sources (environment variables, command-line) override earlier ones.
> - Never store secrets in JSON files that are committed to source control.

---

## Environment-Specific Middleware

One of the most important uses of environments is configuring different middleware pipelines for development versus production.

### The Developer Exception Page

In Development, you want detailed error pages that show stack traces, query strings, headers, and routing information. In Production, you want a generic error page that reveals nothing.

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    // Shows detailed exception info: stack trace, source code, query params, cookies, headers
    app.UseDeveloperExceptionPage();
}
else
{
    // Shows a user-friendly error page with no technical details
    app.UseExceptionHandler("/Error");

    // Adds Strict-Transport-Security header for production HTTPS
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

### Other Environment-Conditional Middleware

```csharp
if (app.Environment.IsDevelopment())
{
    // Swagger UI for API documentation during development
    app.UseSwagger();
    app.UseSwaggerUI(options =>
    {
        options.SwaggerEndpoint("/swagger/v1/swagger.json", "Order API v1");
    });

    // Database error page for EF Core migration issues
    app.UseMigrationsEndPoint();
}

if (!app.Environment.IsDevelopment())
{
    // Response compression is usually handled by a reverse proxy in production,
    // but you might enable it here for staging/production
    app.UseResponseCompression();
}
```

> [!example] Typical Middleware Pipeline by Environment
> **Development**: UseDeveloperExceptionPage, UseSwagger, UseSwaggerUI, UseMigrationsEndPoint
> **Staging**: UseExceptionHandler, UseHsts, UseResponseCompression
> **Production**: UseExceptionHandler, UseHsts, UseResponseCompression

> [!summary] Section Summary
> - `UseDeveloperExceptionPage()` provides rich error details and should only run in Development.
> - `UseExceptionHandler("/Error")` provides a safe, generic error page for Staging and Production.
> - Swagger, migration endpoints, and other development tools should be gated behind `IsDevelopment()`.
> - `UseHsts()` sends the HSTS header and is appropriate for non-Development environments.

---

## Environment-Specific Services

You can register different service implementations based on the environment. This is useful for swapping real services with fakes or mocks during development.

### Example: Email Service

```csharp
var builder = WebApplication.CreateBuilder(args);

if (builder.Environment.IsDevelopment())
{
    // In development, log emails to the console instead of sending them
    builder.Services.AddSingleton<IEmailSender, ConsoleEmailSender>();
}
else
{
    // In staging and production, use the real SMTP sender
    builder.Services.AddSingleton<IEmailSender, SmtpEmailSender>();
}
```

```csharp
public interface IEmailSender
{
    Task SendAsync(string to, string subject, string body);
}

public class ConsoleEmailSender : IEmailSender
{
    public Task SendAsync(string to, string subject, string body)
    {
        Console.WriteLine("========== EMAIL ==========");
        Console.WriteLine($"To: {to}");
        Console.WriteLine($"Subject: {subject}");
        Console.WriteLine($"Body: {body}");
        Console.WriteLine("===========================");
        return Task.CompletedTask;
    }
}

public class SmtpEmailSender : IEmailSender
{
    private readonly EmailSettings _settings;

    public SmtpEmailSender(IOptions<EmailSettings> settings)
    {
        _settings = settings.Value;
    }

    public async Task SendAsync(string to, string subject, string body)
    {
        using var client = new SmtpClient(_settings.SmtpServer, _settings.Port);
        client.EnableSsl = _settings.EnableSsl;
        var message = new MailMessage("noreply@company.com", to, subject, body);
        await client.SendMailAsync(message);
    }
}
```

### Example: Payment Gateway

```csharp
if (builder.Environment.IsDevelopment() || builder.Environment.IsStaging())
{
    // Use a sandbox/mock payment processor
    builder.Services.AddSingleton<IPaymentGateway, SandboxPaymentGateway>();
}
else
{
    // Use the live payment processor
    builder.Services.AddSingleton<IPaymentGateway, StripePaymentGateway>();
}
```

### Example: Logging Configuration

```csharp
if (builder.Environment.IsDevelopment())
{
    builder.Services.AddLogging(logging =>
    {
        logging.AddConsole();
        logging.AddDebug();
        logging.SetMinimumLevel(LogLevel.Debug);
    });
}
else
{
    builder.Services.AddLogging(logging =>
    {
        logging.AddConsole();
        logging.SetMinimumLevel(LogLevel.Warning);
        // Add a structured logging sink for production
        // logging.AddApplicationInsights();
    });
}
```

> [!summary] Section Summary
> - Use `builder.Environment.IsDevelopment()` to register different service implementations per environment.
> - Common pattern: mock/console services in Development, real services in Production.
> - Payment gateways, email senders, and logging are typical candidates for environment-based swapping.

---

## Environment Tag Helper in Razor

Razor Pages and MVC views have a built-in `<environment>` tag helper that conditionally renders HTML based on the current environment.

### Basic Usage

```html
<environment include="Development">
    <!-- Only rendered when ASPNETCORE_ENVIRONMENT is Development -->
    <link rel="stylesheet" href="~/css/site.css" />
    <script src="~/js/site.js"></script>
</environment>

<environment include="Staging,Production">
    <!-- Only rendered in Staging or Production -->
    <link rel="stylesheet" href="~/css/site.min.css" asp-append-version="true" />
    <script src="~/js/site.min.js" asp-append-version="true"></script>
</environment>
```

### Using the exclude Attribute

```html
<environment exclude="Development">
    <!-- Rendered in every environment EXCEPT Development -->
    <link rel="stylesheet"
          href="https://cdn.company.com/css/bootstrap.min.css"
          asp-fallback-href="~/lib/bootstrap/css/bootstrap.min.css"
          asp-fallback-test-class="sr-only"
          asp-fallback-test-property="position"
          asp-fallback-test-value="absolute" />
</environment>
```

### Real-World Example: Debug Toolbar

```html
<environment include="Development">
    <div class="debug-toolbar"
         style="position:fixed; bottom:0; left:0; right:0;
                background:#333; color:#fff; padding:8px; font-size:12px; z-index:9999;">
        <span>Environment: Development</span>
        <span>| Server: @Environment.MachineName</span>
        <span>| Time: @DateTime.Now.ToString("HH:mm:ss")</span>
    </div>
</environment>
```

> [!ad-note] Tag Helper Registration
> The `<environment>` tag helper is available automatically when you include `@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers` in your `_ViewImports.cshtml`. This is already present in the default project templates.

> [!summary] Section Summary
> - The `<environment>` tag helper conditionally renders HTML blocks based on the current environment.
> - Use `include` to specify which environments should render the content.
> - Use `exclude` to render content in all environments except the listed ones.
> - Common use case: unminified assets in Development, CDN/minified assets in Production.

---

## Why You Must Never Run Development in Production

> [!warning] Critical Security Concern
> Running with `ASPNETCORE_ENVIRONMENT=Development` on a production server is a serious security vulnerability. This section explains why.

### Detailed Error Pages Leak Sensitive Information

The Developer Exception Page (`UseDeveloperExceptionPage()`) displays:

- **Full stack traces** with file paths and line numbers, revealing your project structure
- **Source code snippets** around the line that threw the exception
- **Query string values** from the URL
- **HTTP headers**, including authentication tokens and cookies
- **Route data** and endpoint information
- **Environment variables** if the error occurs during startup

An attacker who triggers an error on a Development-mode production server can learn:

- The server's file system layout
- The .NET version and framework dependencies
- Database connection details from exception messages
- Internal API structures from routing information
- Authentication mechanisms from header inspection

### Verbose Logging Fills Disks and Exposes Data

Development-level logging (`LogLevel.Debug` or `LogLevel.Trace`) writes an enormous volume of data. In production, this:

- Consumes disk space rapidly
- Degrades performance due to I/O overhead
- May log sensitive user data (request bodies, form values, query parameters)
- Makes it harder to find genuine errors in the noise

### Development Services Are Not Production-Ready

If you conditionally register mock services for Development (as shown in the services section above), running Development in production means:

- Emails go to the console instead of real recipients
- Payment processing hits sandbox APIs instead of live payment systems
- Authentication might use relaxed validation

### The Correct Pattern

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// This pattern ensures production-safe defaults
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}
```

> [!tip] Verify Your Production Environment
> Add a startup check to confirm the environment is not Development when deploying:
> ```csharp
> if (app.Environment.IsDevelopment())
> {
>     app.Logger.LogWarning("Application is running in Development mode!");
> }
> ```
> Some teams add CI/CD pipeline checks that fail the deployment if the environment is set to Development.

> [!summary] Section Summary
> - The Developer Exception Page leaks stack traces, source code, headers, and environment variables.
> - Verbose logging consumes resources and may expose sensitive user data.
> - Mock services registered for Development will not function correctly in production.
> - Always verify your production deployments are not running in Development mode.

---

## Custom Environments

You are not limited to the three built-in environment names. You can create any custom environment by setting `ASPNETCORE_ENVIRONMENT` to a custom string.

### Common Custom Environments

| Environment | Use Case                                                        |
| ----------- | --------------------------------------------------------------- |
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

---

## Environment Variables vs Command-Line Arguments

Both environment variables and command-line arguments can configure the application, but they serve different roles.

### Environment Variables

```bash
# Set the environment
export ASPNETCORE_ENVIRONMENT=Production

# Override configuration values using double-underscore separator
export ConnectionStrings__InventoryDb="Server=prod-server;Database=InventoryDb;..."
export EmailSettings__SmtpServer="smtp.production.com"
```

> [!ad-note] Double Underscore Convention
> In environment variables, the `__` (double underscore) replaces the `:` section separator used in JSON configuration. So `ConnectionStrings:InventoryDb` in JSON becomes `ConnectionStrings__InventoryDb` as an environment variable.

### Command-Line Arguments

```bash
dotnet run --environment Staging --urls "https://localhost:7200"
dotnet OrderService.dll --ConnectionStrings:InventoryDb="Server=staging-server;..."
```

### When to Use Each

| Approach              | Best For                                       |
| --------------------- | ---------------------------------------------- |
| Environment variables | Server deployments, Docker containers, CI/CD   |
| Command-line args     | Quick local testing, overriding a single value  |
| launchSettings.json   | Standard local development workflow             |
| appsettings.*.json    | Structured, version-controlled configuration    |

> [!summary] Section Summary
> - Environment variables use `__` as a section separator and are ideal for server deployments.
> - Command-line arguments are convenient for quick local overrides.
> - Both override values from `appsettings.json` files, with command-line having the highest priority.

---

## Practical Patterns and Real-World Examples

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

---

## Comprehensive Summary

> [!tip] Complete Summary
> ASP.NET Core's environment system is the foundation for building applications that behave differently across Development, Staging, and Production. The environment is set via the `ASPNETCORE_ENVIRONMENT` environment variable (or `launchSettings.json` during local development) and defaults to Production when unset. Each environment can load its own `appsettings.{Environment}.json` configuration file, which overrides the base `appsettings.json`. In code, you check the environment using `IsDevelopment()`, `IsStaging()`, `IsProduction()`, or `IsEnvironment("Custom")` to conditionally register services, configure middleware, and control behavior. The Developer Exception Page should only run in Development because it leaks stack traces, source code, and headers. The `<environment>` Razor tag helper lets you conditionally render HTML (such as minified vs unminified assets) based on the current environment. You can extend beyond the three built-in names by creating custom environments like QA or UAT. The key principle is defense in depth: default to the most restrictive mode (Production), be explicit about Development-only features, and never expose diagnostic tools to end users.

---

## Related Topics

- [[ASP.NET Core Overview]]
- [[Project Structure]]
- [[Hosting Model]]
- [[Program.cs and Startup]]
- [[Configuration and Options Pattern]]
- [[Logging in ASP.NET Core]]
- [[Middleware Pipeline]]
- [[Error Handling and Exception Pages]]
- [[Dependency Injection in ASP.NET Core]]
- [[User Secrets]]
