---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - lifetimes
---

## Demonstrating Lifetime Behavior

The best way to understand lifetimes is to see them in action. Here is a complete example that makes the differences visible.

### The Service

```csharp
public interface ILifetimeReporter
{
    Guid InstanceId { get; }
    string Lifetime { get; }
}

public class TransientReporter : ILifetimeReporter
{
    public Guid InstanceId { get; } = Guid.NewGuid();
    public string Lifetime => "Transient";
}

public class ScopedReporter : ILifetimeReporter
{
    public Guid InstanceId { get; } = Guid.NewGuid();
    public string Lifetime => "Scoped";
}

public class SingletonReporter : ILifetimeReporter
{
    public Guid InstanceId { get; } = Guid.NewGuid();
    public string Lifetime => "Singleton";
}
```

### Registration

```csharp
builder.Services.AddTransient<TransientReporter>();
builder.Services.AddScoped<ScopedReporter>();
builder.Services.AddSingleton<SingletonReporter>();
builder.Services.AddTransient<ChildService>();
```

### A Child Service (to show same-request behavior)

```csharp
public class ChildService
{
    public TransientReporter Transient { get; }
    public ScopedReporter Scoped { get; }
    public SingletonReporter Singleton { get; }

    public ChildService(
        TransientReporter transient,
        ScopedReporter scoped,
        SingletonReporter singleton)
    {
        Transient = transient;
        Scoped = scoped;
        Singleton = singleton;
    }
}
```

### The Controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class LifetimeController : ControllerBase
{
    private readonly TransientReporter _transient;
    private readonly ScopedReporter _scoped;
    private readonly SingletonReporter _singleton;
    private readonly ChildService _childService;

    public LifetimeController(
        TransientReporter transient,
        ScopedReporter scoped,
        SingletonReporter singleton,
        ChildService childService)
    {
        _transient = transient;
        _scoped = scoped;
        _singleton = singleton;
        _childService = childService;
    }

    [HttpGet]
    public IActionResult Get()
    {
        return Ok(new
        {
            Controller = new
            {
                Transient = _transient.InstanceId,
                Scoped = _scoped.InstanceId,
                Singleton = _singleton.InstanceId
            },
            ChildService = new
            {
                Transient = _childService.Transient.InstanceId,
                Scoped = _childService.Scoped.InstanceId,
                Singleton = _childService.Singleton.InstanceId
            }
        });
    }
}
```

### Expected Output

**First request:**

```json
{
  "controller": {
    "transient": "a1b2c3d4-...",
    "scoped":    "e5f6a7b8-...",
    "singleton": "11111111-..."
  },
  "childService": {
    "transient": "c9d0e1f2-...",   // DIFFERENT from controller (new instance)
    "scoped":    "e5f6a7b8-...",   // SAME as controller (shared per request)
    "singleton": "11111111-..."    // SAME as controller (shared globally)
  }
}
```

**Second request:**

```json
{
  "controller": {
    "transient": "f3a4b5c6-...",   // DIFFERENT from first request
    "scoped":    "d7e8f9a0-...",   // DIFFERENT from first request (new scope)
    "singleton": "11111111-..."    // SAME as first request (app-wide)
  },
  "childService": {
    "transient": "b1c2d3e4-...",   // DIFFERENT from everything
    "scoped":    "d7e8f9a0-...",   // SAME as controller within this request
    "singleton": "11111111-..."    // SAME as always
  }
}
```

> [!example]
> Key observations from the output:
> - **Transient**: All four Guid values across both requests are different. Every injection creates a new instance.
> - **Scoped**: Within a single request, the controller and child service share the same Guid. Across requests, the Guid changes.
> - **Singleton**: The same Guid appears everywhere, in both requests. It never changes until the app restarts.

> [!summary] Section Summary
> - A service that generates a Guid in its constructor makes lifetime behavior directly observable.
> - Transient: every injection point gets a unique Guid, even within the same request.
> - Scoped: the same Guid is shared across all injection points within one request, but changes per request.
> - Singleton: one Guid for the entire application lifetime, shared everywhere.
> - This pattern is an excellent debugging technique when you suspect lifetime misconfiguration.
