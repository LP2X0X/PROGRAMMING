---
tags:
  - csharp
  - asp-net-core
  - logging
  - ilogger
  - observability
---


**Log scopes** add contextual properties to all log entries within a block of code. This is invaluable for tracing a request through multiple service calls.

```csharp
public async Task<Order> ProcessOrderAsync(int orderId, int customerId)
{
    // All log entries within this using block will include
    // OrderId and CustomerId as properties
    using (_logger.BeginScope(
        new Dictionary<string, object>
        {
            ["OrderId"] = orderId,
            ["CustomerId"] = customerId
        }))
    {
        _logger.LogInformation("Starting order processing");
        // Log entry includes: OrderId=42, CustomerId=567

        await ValidateInventory();
        // Any logs inside ValidateInventory also include OrderId and CustomerId

        await ProcessPayment();
        // Same here -- the scope follows the async execution context

        _logger.LogInformation("Order processing completed");
    }
    // Scope ends here -- OrderId and CustomerId are no longer added
}
```

You can also use simpler string-based scopes:

```csharp
using (_logger.BeginScope("Processing order {OrderId}", orderId))
{
    _logger.LogInformation("Validating inventory");
    _logger.LogInformation("Processing payment");
}
```

## Enabling Scopes in Configuration

For the Console provider, scopes are disabled by default. Enable them:

```json
{
  "Logging": {
    "Console": {
      "IncludeScopes": true
    }
  }
}
```

> [!ad-note]
> Serilog automatically includes scope properties when you use the `FromLogContext` enricher (which you should always include). No additional configuration is needed.

> [!summary] Section Summary
> - `BeginScope` adds contextual properties to all log entries within the `using` block
> - Scopes follow async execution -- they work correctly across `await` boundaries
> - Use dictionary scopes for named properties or string scopes for simple context
> - Console provider requires `"IncludeScopes": true`; Serilog handles scopes automatically via `FromLogContext`
