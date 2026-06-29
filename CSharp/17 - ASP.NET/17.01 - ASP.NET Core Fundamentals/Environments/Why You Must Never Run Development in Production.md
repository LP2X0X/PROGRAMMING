---
tags: [csharp, asp-net-core, environments, configuration]
---


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
