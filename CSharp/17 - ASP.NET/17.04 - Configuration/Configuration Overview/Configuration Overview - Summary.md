---
tags:
  - csharp
  - asp-net-core
  - configuration
  - fundamentals
---


> [!tip] Complete Summary
> **The ASP.NET Core configuration system** is a provider-based, layered architecture where multiple sources contribute key-value pairs and later sources override earlier ones.
>
> **Default provider order** (low to high priority):
> 1. `appsettings.json` -- base defaults
> 2. `appsettings.{Environment}.json` -- environment-specific overrides
> 3. User secrets -- development-only sensitive values
> 4. Environment variables -- deployment/container config
> 5. Command-line arguments -- highest priority, one-off overrides
>
> **Key interfaces:**
> - `IConfiguration` -- the primary abstraction for reading values; inject this into services
> - `IConfigurationRoot` -- extends `IConfiguration` with provider inspection and reload; use at startup for diagnostics
>
> **Reading values:**
> - Indexer: `config["Section:Key"]` returns `string?`
> - Typed: `config.GetValue<int>("Key", defaultValue)` with type conversion
> - Sections: `config.GetSection("Group")` for scoped access
> - Connection strings: `config.GetConnectionString("Name")` as a shortcut
>
> **Best practices:**
> - Never commit secrets to JSON files
> - Use `GetDebugView()` to troubleshoot configuration resolution
> - Prefer the [[Options Pattern]] for strongly typed settings
> - Be aware of provider order when adding custom sources
> - Fail fast on missing critical configuration (connection strings, required API keys)
