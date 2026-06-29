---
tags:
  - csharp
  - asp-net-core
  - configuration
  - fundamentals
---


> [!abstract] Overview
> ASP.NET Core replaces the monolithic `web.config` / `ConfigurationManager` approach with a modern, **layered, provider-based configuration system**. Multiple sources (JSON files, environment variables, command-line arguments, user secrets, and more) are stacked in a defined order where **later sources override earlier ones**. The entire system is built around the `IConfiguration` interface and integrates seamlessly with dependency injection, the Options pattern, and environment-based overrides.

> [!info] Definition
> The **configuration system** in ASP.NET Core is a unified, extensible framework for reading application settings from multiple sources at startup and making them available throughout the application via dependency injection.

The core ideas are:

- **Provider-based** -- each source of configuration (JSON file, environment variable, etc.) has a corresponding **configuration provider** that knows how to read key-value pairs from that source.
- **Layered / override semantics** -- providers are registered in a specific order. When two providers supply the same key, the **last one registered wins**.
- **Flat key space with hierarchy** -- all configuration is normalized into a flat dictionary of `string` keys and `string` values. Hierarchical structure is expressed with a colon delimiter (e.g., `Smtp:Host`).
- **No special XML format** -- unlike the old `web.config`, configuration files are plain JSON (by default), and the system is format-agnostic.

```
appsettings.json          -->  base settings
appsettings.Production.json --> environment override
Environment variables     -->  deployment override
Command-line args         -->  one-off override
```

Each layer can override any key from the layers below it without modifying those files.

> [!summary] Section Summary
> ASP.NET Core configuration is a layered, provider-based system that replaces `web.config`. Multiple sources are stacked so that later sources override earlier ones, giving you flexible, environment-aware settings with no XML ceremony.
