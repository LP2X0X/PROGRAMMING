---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - exceptions
  - middleware
---


This principle is important enough to have its own section. **Never** return exception messages, stack traces, inner exceptions, or implementation details in production error responses.

## What Gets Exposed Accidentally

```csharp
// DANGEROUS: Exposing the raw exception message
catch (Exception ex)
{
    return StatusCode(500, new { error = ex.Message });
}
// ex.Message might be:
// "Invalid column name 'Proce'. (SqlException)"
// "Access denied for user 'appuser'@'10.0.1.42' to database 'products'"
// "Object reference not set to an instance of an object."
```

These messages reveal:
- Database column names and schema
- Internal IP addresses and usernames
- Database technology (SQL Server, MySQL, etc.)
- That you have null reference bugs (code quality signal)
- File system paths in `FileNotFoundException`

## The Safe Pattern

```csharp
// SAFE: Generic message for production, detailed for development
var detail = _environment.IsDevelopment()
    ? ex.ToString()    // Full exception with stack trace for dev
    : "An unexpected error occurred. Please contact support with " +
      $"reference ID: {context.TraceIdentifier}";

// The traceId lets support find the full exception in server logs
// without exposing internals to the client
```

> [!danger]
> Even seemingly harmless exception messages can reveal too much. `"The ConnectionString property has not been initialized"` tells an attacker you use ADO.NET and have a configuration issue. `"No such host is known: payments.internal.corp.com"` reveals your internal service naming convention. Always use generic messages in production and log the full details server-side.

> [!summary] Section Summary
> - Never return `ex.Message`, `ex.StackTrace`, or `ex.ToString()` in production responses
> - Use a trace ID / correlation ID so users can reference the error when contacting support
> - Log the full exception details server-side where only your team can see them
> - Even "harmless" messages can leak database schema, internal hostnames, usernames, and technology choices
