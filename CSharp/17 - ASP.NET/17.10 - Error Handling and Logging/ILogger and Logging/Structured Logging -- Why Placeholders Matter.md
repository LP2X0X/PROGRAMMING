---
tags:
  - csharp
  - asp-net-core
  - logging
  - ilogger
  - observability
---


This is one of the most important concepts in modern logging. **Structured logging** means log entries are not just text strings -- they are data records with named properties that can be searched, filtered, and aggregated.

## The Wrong Way -- String Interpolation

```csharp
// BAD: String interpolation
_logger.LogInformation($"Processing order {orderId} for customer {customerId}");
```

This produces a flat string: `"Processing order 42 for customer 567"`. The logging system sees it as a single pre-formatted message with no structure. You cannot search for "all logs where orderId = 42" because `orderId` is just part of a text string.

## The Right Way -- Message Templates

```csharp
// GOOD: Message template with named placeholders
_logger.LogInformation("Processing order {OrderId} for customer {CustomerId}",
    orderId, customerId);
```

This produces a log entry with:
- A **message template**: `"Processing order {OrderId} for customer {CustomerId}"`
- Named **properties**: `OrderId = 42`, `CustomerId = 567`
- A **rendered message**: `"Processing order 42 for customer 567"`

In a log aggregation system like Seq, Elasticsearch, or Application Insights, you can now query: `OrderId = 42` and find every log entry related to that order -- across all services, all log levels.

## How It Works Under the Hood

The logging framework parses the message template at call time. Each `{Placeholder}` becomes a named property in the log event. The values are passed as parameters (not interpolated into the string) and are stored as structured data alongside the rendered text.

```csharp
// The placeholder names become property names in the log event
_logger.LogInformation(
    "User {UserName} from {IpAddress} accessed {Endpoint} in {ElapsedMs}ms",
    userName,       // Property: UserName
    ipAddress,      // Property: IpAddress
    endpoint,       // Property: Endpoint
    elapsedMs       // Property: ElapsedMs
);

// In Seq or Elasticsearch, you can now query:
// UserName = "john.doe" AND ElapsedMs > 1000
// IpAddress = "10.0.1.42"
// Endpoint = "/api/orders" AND @Level = "Warning"
```

## Destructuring with @

For complex objects, prefix the placeholder with `@` to serialize the object's properties instead of calling `.ToString()`:

```csharp
var order = new { Id = 42, CustomerId = 567, Total = 99.99 };

// Without @: logs order.ToString() which is "{ Id = 42, CustomerId = 567, Total = 99.99 }"
_logger.LogInformation("Processing order {Order}", order);

// With @: destructures the object into individual properties
_logger.LogInformation("Processing order {@Order}", order);
// Creates properties: Order.Id = 42, Order.CustomerId = 567, Order.Total = 99.99
```

> [!danger]
> ==Never use string interpolation (`$"..."`) with logging methods.== Besides losing structure, interpolated strings are always formatted -- even if the log level is disabled. This wastes CPU and allocations. With message templates, the formatting only happens if the log level is enabled.

> [!tip]
> Adopt consistent naming conventions for your log properties across the entire application. Always use `{OrderId}`, not sometimes `{orderId}` and sometimes `{order_id}`. Consistent names make cross-service log correlation possible.

> [!summary] Section Summary
> - Structured logging uses **message templates** with `{NamedPlaceholders}` instead of string interpolation
> - Named placeholders become searchable, filterable properties in log aggregation systems
> - Use `@` prefix to destructure complex objects into their individual properties
> - Never use `$"..."` string interpolation -- it destroys structure and wastes resources when the level is disabled
> - Consistent property naming across services enables powerful cross-service log correlation
