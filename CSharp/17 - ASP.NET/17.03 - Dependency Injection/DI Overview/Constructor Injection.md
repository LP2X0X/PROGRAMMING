---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - fundamentals
---

## Constructor Injection

Constructor injection is the **primary and recommended** DI pattern in ASP.NET Core. The idea is simple: a class declares its dependencies as constructor parameters, and the DI container provides them automatically when it creates an instance.

```csharp
public class CustomerController : ControllerBase
{
    private readonly ICustomerService _customerService;
    private readonly ILogger<CustomerController> _logger;

    // Constructor injection -- the framework provides these automatically
    public CustomerController(
        ICustomerService customerService,
        ILogger<CustomerController> logger)
    {
        _customerService = customerService;
        _logger = logger;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetCustomer(int id)
    {
        _logger.LogInformation("Fetching customer {CustomerId}", id);
        var customer = await _customerService.GetByIdAsync(id);

        if (customer is null)
            return NotFound();

        return Ok(customer);
    }
}
```

### Why Constructor Injection Is Preferred

| Advantage | Explanation |
|---|---|
| **Explicit dependencies** | Every dependency is visible in the constructor signature |
| **Immutability** | Dependencies are assigned to `readonly` fields, preventing reassignment |
| **Required by default** | If a required service is not registered, the app fails at startup with a clear error, not at runtime |
| **Framework support** | ASP.NET Core's built-in container resolves constructor parameters automatically |

> [!warning] Common Misconception
> You might think you need to "wire up" each controller manually, passing in the correct service instances. You do not. The DI container reads the constructor parameters, looks up the registered services, creates the necessary instances, and passes them in. You only need to register the services once at startup.

> [!ad-note]
> ASP.NET Core also supports **method injection** via the `[FromServices]` attribute on action method parameters, and **service location** via `HttpContext.RequestServices.GetService<T>()`. However, constructor injection should be your default choice. Method injection is useful for dependencies needed by only one action method. Service location (the Service Locator pattern) is generally considered an anti-pattern because it hides dependencies.

> [!summary] Section Summary
> - Constructor injection is the primary DI pattern in ASP.NET Core -- declare dependencies as constructor parameters.
> - Dependencies should be stored in `private readonly` fields for immutability.
> - The DI container resolves all constructor parameters automatically; no manual wiring is needed.
> - Constructor injection makes dependencies explicit, required, and visible.
> - Prefer constructor injection over method injection or the Service Locator pattern.
