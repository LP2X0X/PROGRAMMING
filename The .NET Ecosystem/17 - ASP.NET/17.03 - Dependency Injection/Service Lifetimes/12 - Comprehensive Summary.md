---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - lifetimes
---

## Comprehensive Summary

> [!tip] Complete Summary
> **The Three Lifetimes:**
> - **Transient** (`AddTransient`) -- new instance per injection. Use for stateless, lightweight services like validators and formatters.
> - **Scoped** (`AddScoped`) -- one instance per HTTP request (per scope). Use for DbContext, repositories, Unit of Work, and anything that needs request-level isolation.
> - **Singleton** (`AddSingleton`) -- one instance for the entire application. Use for caches, configuration, and shared infrastructure. Must be thread-safe.
>
> **Key Rules:**
> - A service can only safely depend on services with an **equal or longer lifetime** (Transient < Scoped < Singleton).
> - Violating this rule creates a **captive dependency** -- a scoped service trapped inside a singleton, leading to stale state, memory leaks, and thread-safety bugs.
> - Use `IServiceScopeFactory` when a singleton needs access to scoped services.
> - `DbContext` should always be Scoped (EF Core's default). `HttpClient` should always go through `IHttpClientFactory`.
>
> **Safety Nets:**
> - `ValidateScopes` catches captive dependencies at runtime (enabled in Development by default).
> - `ValidateOnBuild` catches missing registrations at startup (enabled in Development by default).
> - Consider enabling `ValidateOnBuild` in all environments for early failure detection.
>
> **When in Doubt:** Start with **Scoped**. It is the safest default for business services -- it prevents cross-request leaks (unlike Singleton) and avoids unnecessary instance creation (unlike Transient).

---

## Related Topics

- [[DI Overview]] -- fundamentals of dependency injection in ASP.NET Core
- [[Registration Patterns]] -- `Add*` methods, factory registrations, keyed services, open generics
- [[Common DI Pitfalls]] -- anti-patterns including service locator, captive dependencies, and disposal mistakes
- [[Entity Framework Core]] -- DbContext configuration and lifetime management
- [[Background Services]] -- implementing `IHostedService` and `BackgroundService` with proper scope management
- [[Middleware Pipeline]] -- how ASP.NET Core creates per-request scopes automatically
- [[IHttpClientFactory]] -- managing HTTP connections with proper lifetime handling

---

## Further Reading

- [Microsoft Docs: Dependency injection in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection)
- [Microsoft Docs: Service lifetimes](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection#service-lifetimes)
- [Microsoft Docs: IHttpClientFactory](https://learn.microsoft.com/en-us/dotnet/core/extensions/httpclient-factory)
- [Microsoft Docs: Background tasks with hosted services](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/host/hosted-services)
- [Andrew Lock: The dangers of the captive dependency](https://andrewlock.net/the-dangers-of-the-captive-dependency-in-asp-net-core/)
- [Steve Collins: ASP.NET Core Dependency Injection Best Practices](https://stevetalkscode.co.uk/dependency-injection-best-practices)
