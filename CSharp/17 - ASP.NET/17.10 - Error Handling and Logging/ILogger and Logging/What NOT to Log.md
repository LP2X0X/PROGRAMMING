---
tags:
  - csharp
  - asp-net-core
  - logging
  - ilogger
  - observability
---


Logging sensitive data is a security and compliance violation. Your logs are stored in log files, log aggregation services, and potentially backed up -- all of which expand the attack surface.

## Never Log These

| Data Type | Why It Is Dangerous |
|---|---|
| **Passwords / secrets** | Even hashed passwords should not be in logs; plain text is catastrophic |
| **Authentication tokens** | JWT tokens, API keys, session tokens -- anyone with log access can impersonate users |
| **Credit card numbers** | PCI-DSS compliance violation; can result in fines and loss of payment processing |
| **Social Security Numbers** | PII laws (GDPR, CCPA) make this a compliance violation |
| **Medical records** | HIPAA violation (if applicable) |
| **Full request/response bodies** | May contain any of the above in POST requests or API responses |
| **Connection strings** | Contain database credentials |
| **Encryption keys** | Obvious compromise |

## Defensive Patterns

```csharp
// BAD: Logging the full request body
_logger.LogInformation("Received: {Body}", await ReadBodyAsync(context));

// BAD: Logging authentication headers
_logger.LogDebug("Auth header: {Auth}", context.Request.Headers.Authorization);

// BAD: Logging user input that might contain sensitive data
_logger.LogInformation("User submitted: {FormData}", request.ToString());

// GOOD: Log only safe, identifying information
_logger.LogInformation(
    "Login attempt for user {Username} from {IpAddress}",
    request.Username,  // Username is safe to log
    context.Connection.RemoteIpAddress);

// GOOD: Mask sensitive data if you must log it for debugging
_logger.LogDebug(
    "Processing card ending in {CardLast4} for {Amount:C}",
    cardNumber[^4..],  // Only last 4 digits
    amount);
```

> [!danger]
> Do not rely on log level filtering to protect sensitive data. "We only log card numbers at Trace level, and Trace is disabled in production" is not a security measure. Log levels can be changed at runtime, and a developer troubleshooting an issue might enable Trace without realizing what gets exposed. ==Never write sensitive data to log statements at any level.==

## Serilog Destructure Policies

Serilog can automatically mask sensitive properties during destructuring:

```csharp
builder.Host.UseSerilog((context, configuration) =>
{
    configuration
        .Destructure.ByTransforming<LoginRequest>(
            r => new { r.Username, Password = "***" })
        .Destructure.ByTransforming<PaymentRequest>(
            r => new { r.OrderId, r.Amount, CardNumber = "***" });
});
```

> [!summary] Section Summary
> - Never log passwords, tokens, credit cards, SSNs, connection strings, or encryption keys at any level
> - Do not rely on log level filtering as a security measure -- levels can change at runtime
> - Mask sensitive fields if partial information is needed (e.g., last 4 digits of a card)
> - Use Serilog's destructure policies to automatically redact sensitive properties from objects
> - Logging full request/response bodies is dangerous because they may contain any sensitive data
