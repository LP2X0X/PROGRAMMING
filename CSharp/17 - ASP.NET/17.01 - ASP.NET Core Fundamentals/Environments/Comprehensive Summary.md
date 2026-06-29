---
tags: [csharp, asp-net-core, environments, configuration]
---


> [!tip] Complete Summary
> ASP.NET Core's environment system is the foundation for building applications that behave differently across Development, Staging, and Production. The environment is set via the `ASPNETCORE_ENVIRONMENT` environment variable (or `launchSettings.json` during local development) and defaults to Production when unset. Each environment can load its own `appsettings.{Environment}.json` configuration file, which overrides the base `appsettings.json`. In code, you check the environment using `IsDevelopment()`, `IsStaging()`, `IsProduction()`, or `IsEnvironment("Custom")` to conditionally register services, configure middleware, and control behavior. The Developer Exception Page should only run in Development because it leaks stack traces, source code, and headers. The `<environment>` Razor tag helper lets you conditionally render HTML (such as minified vs unminified assets) based on the current environment. You can extend beyond the three built-in names by creating custom environments like QA or UAT. The key principle is defense in depth: default to the most restrictive mode (Production), be explicit about Development-only features, and never expose diagnostic tools to end users.
