---
tags:
  - csharp
  - asp-net-core
  - logging
  - ilogger
  - observability
---


ASP.NET Core defines six log levels, ordered from most verbose (Trace) to most severe (Critical). Choosing the right level is an art that directly affects the usefulness of your logs.

| Level | Numeric | When to Use | Example |
|---|---|---|---|
| **Trace** | 0 | Highly detailed diagnostic info; may contain sensitive data | `"Entering method GetProduct with id=42"` |
| **Debug** | 1 | Internal application flow useful during development | `"Cache miss for key 'product:42', fetching from DB"` |
| **Information** | 2 | General application flow -- significant business events | `"Order 1234 created for customer 567"` |
| **Warning** | 3 | Unexpected events that do not stop the application | `"Payment retry attempt 2 of 3 for order 1234"` |
| **Error** | 4 | An operation failed but the application continues | `"Failed to send confirmation email for order 1234"` |
| **Critical** | 5 | Application-wide failure requiring immediate attention | `"Database connection pool exhausted, no connections available"` |
| **None** | 6 | Disables logging entirely for a category (configuration only) | Used only in config, not in code |

## Detailed Guidance

**Trace** -- Use for method entry/exit, parameter values, and detailed diagnostic information. This level generates enormous volume and is typically only enabled when actively debugging a specific issue. Never log sensitive data even at Trace level if logs are centralized.

**Debug** -- Use for internal logic decisions: cache hits/misses, branch decisions, computed values. Useful during development and for diagnosing specific issues in production (by temporarily lowering the level for a category).

**Information** -- Use for significant business events: user logged in, order created, payment processed, report generated. These form the "story" of what your application is doing. In production, this is typically the default minimum level.

**Warning** -- Use when something unexpected happened but the application recovered or can continue. Retry attempts, deprecated API usage, configuration fallbacks, slow queries.

**Error** -- Use when an operation failed. The specific request could not be completed, but the application can still serve other requests. Include the exception object when available.

**Critical** -- Use for catastrophic failures: database unreachable, out of memory, unrecoverable state corruption. These should trigger immediate alerts. Most applications log Critical very rarely.

```csharp
public async Task<Order> ProcessOrderAsync(OrderRequest request)
{
    _logger.LogTrace("Entering ProcessOrderAsync with {@Request}", request);

    _logger.LogDebug("Checking inventory for {ProductCount} products", 
        request.Items.Count);

    if (await _inventory.ReserveAsync(request.Items))
    {
        _logger.LogInformation(
            "Order {OrderId} placed by customer {CustomerId} for {Total:C}",
            request.OrderId, request.CustomerId, request.Total);
    }
    else
    {
        _logger.LogWarning(
            "Insufficient inventory for order {OrderId}, " +
            "customer {CustomerId} will be notified",
            request.OrderId, request.CustomerId);
    }

    try
    {
        await _emailService.SendConfirmationAsync(request.Email);
    }
    catch (SmtpException ex)
    {
        // The order succeeded, but email failed -- not fatal
        _logger.LogError(ex,
            "Failed to send confirmation email for order {OrderId} " +
            "to {Email}",
            request.OrderId, request.Email);
    }

    _logger.LogTrace("Exiting ProcessOrderAsync");
    return order;
}
```

> [!warning] Common Misconception
> Many developers use `LogError` for any exception they catch, including expected business scenarios like "product not found" or "invalid input." These are **not errors** -- they are expected application behavior. Use Warning for expected-but-notable situations and reserve Error for genuine failures. Over-using Error causes alert fatigue and makes real errors harder to find.

> [!summary] Section Summary
> - Six log levels from Trace (most verbose) to Critical (most severe) control the verbosity of your logs
> - Information is the standard production level -- it captures the "story" of business events
> - Warning is for recoverable unexpected situations; Error is for operation failures
> - Critical is rare and should trigger immediate alerts
> - Choose levels carefully: over-logging at Error/Critical causes alert fatigue; under-logging at Information means missing context when debugging
