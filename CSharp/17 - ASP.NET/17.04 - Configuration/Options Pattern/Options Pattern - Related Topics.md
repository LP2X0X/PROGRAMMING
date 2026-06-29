---
tags:
  - csharp
  - asp-net-core
  - configuration
  - options-pattern
---


- [[Configuration Overview]] -- How the configuration system works end-to-end (providers, layering, precedence)
- [[Secrets and Environment Variables]] -- Managing sensitive settings outside of `appsettings.json`
- [[Strongly Typed Configuration]] -- Deeper dive into binding complex types, arrays, and dictionaries
- [[Dependency Injection]] -- The DI container that resolves `IOptions<T>` and its variants
- [[Middleware]] -- Where scoped options (`IOptionsSnapshot<T>`) get their per-request lifetime
- [[Background Services]] -- Primary consumer of `IOptionsMonitor<T>` for long-running tasks

---


- [Microsoft Docs: Options pattern in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options)
- [Microsoft Docs: Configuration in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/)
- [Andrew Lock: Adding validation to strongly typed configuration objects](https://andrewlock.net/adding-validation-to-strongly-typed-configuration-objects-in-asp-net-core/)
- [Steve Smith: ASP.NET Core Options Pattern](https://ardalis.com/aspnetcore-options-pattern/)
