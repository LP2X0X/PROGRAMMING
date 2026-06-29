---
tags: [csharp, asp-net-core, environments, configuration]
---


ASP.NET Core recognizes three environment names by convention. These are not hard-coded restrictions -- they are simply the names the framework provides helper methods for.

| Environment   | Purpose                                                                 | Typical Behavior                                  |
|---|---|---|
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
