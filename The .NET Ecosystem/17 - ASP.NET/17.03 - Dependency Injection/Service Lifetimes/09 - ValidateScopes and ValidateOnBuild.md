---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - lifetimes
---

## ValidateScopes and ValidateOnBuild

ASP.NET Core provides two built-in safety nets to catch lifetime misconfigurations early, both enabled by default in the Development environment.

### ValidateScopes

`ValidateScopes` detects captive dependencies at runtime. When enabled, the container throws an `InvalidOperationException` if you try to resolve a scoped service from the root provider (which is what happens when a singleton captures a scoped service).

```csharp
var builder = WebApplication.CreateBuilder(args);

// Enabled by default in Development. To enable explicitly:
builder.Host.UseDefaultServiceProvider(options =>
{
    options.ValidateScopes = true;
});
```

When a captive dependency is detected, you see an error like:

```
System.InvalidOperationException: Cannot resolve scoped service
'IOrderRepository' from root provider. This is caused by a singleton
service 'IOrderNotificationService' depending on a scoped service.
```

> [!caution]
> `ValidateScopes` is disabled in Production by default for performance reasons. This means captive dependency bugs can slip through to production if you do not test thoroughly in Development. Consider enabling it in staging environments as well.

### ValidateOnBuild

`ValidateOnBuild` checks all service registrations at application startup, before any requests are served. It catches issues like:

- Missing registrations (a service depends on an interface that was never registered)
- Open generic registration errors
- Invalid lifetime combinations (captive dependencies)

```csharp
builder.Host.UseDefaultServiceProvider(options =>
{
    options.ValidateScopes = true;
    options.ValidateOnBuild = true;
});
```

A validation failure at startup looks like:

```
System.AggregateException: Some services are not able to be constructed
(Error while validating the service descriptor
'ServiceType: IOrderNotificationService
Lifetime: Singleton
ImplementationType: OrderNotificationService':
Cannot consume scoped service 'IOrderRepository' from singleton
'OrderNotificationService'.)
```

### Recommended Configuration

```csharp
var builder = WebApplication.CreateBuilder(args);

// The default behavior -- both enabled in Development
// This is what CreateBuilder does internally:
if (builder.Environment.IsDevelopment())
{
    builder.Host.UseDefaultServiceProvider(options =>
    {
        options.ValidateScopes = true;
        options.ValidateOnBuild = true;
    });
}

// For maximum safety in all environments (slight startup cost):
builder.Host.UseDefaultServiceProvider(options =>
{
    options.ValidateScopes = builder.Environment.IsDevelopment();
    options.ValidateOnBuild = true; // Always validate at startup
});
```

> [!tip]
> Even though `ValidateScopes` has a runtime performance cost, `ValidateOnBuild` only runs once at startup. There is almost no reason to disable `ValidateOnBuild` in production -- the one-time startup cost is negligible compared to the debugging pain of a missing registration error at 3 AM.

> [!warning] Common Misconception
> "If `ValidateScopes` does not throw, my lifetimes are all correct." Not necessarily. `ValidateScopes` only catches scoped services resolved from the root provider. If you manually create a scope and resolve services from it, `ValidateScopes` does not check those paths. It is a safety net, not a complete verification.

> [!summary] Section Summary
> - `ValidateScopes` detects captive dependencies at runtime by throwing when scoped services are resolved from the root provider.
> - `ValidateOnBuild` validates all registrations at startup, catching missing dependencies before any request is served.
> - Both are enabled by default in Development via `WebApplication.CreateBuilder`.
> - `ValidateOnBuild` has minimal startup cost and is worth enabling in all environments.
> - These validations are safety nets, not complete guarantees -- thorough testing is still essential.
