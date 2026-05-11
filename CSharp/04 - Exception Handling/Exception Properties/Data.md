---
tags:
 - csharp
 - exception-handling
---

## `Exception.Data`

The `Data` property returns an `IDictionary` collection that lets you attach arbitrary key-value metadata to an exception. This is one of the most underused yet powerful properties for diagnosing issues in production.

### Basic Usage

```csharp
try
{
    ProcessOrder(orderId, customerId);
}
catch (Exception ex)
{
    ex.Data["OrderId"] = orderId;
    ex.Data["CustomerId"] = customerId;
    ex.Data["Timestamp"] = DateTime.UtcNow;
    throw; // re-throw with enriched context
}
```

### Using Object Initializer Syntax

```csharp
throw new InvalidOperationException($"{PetName} has overheated!")
{
    HelpLink = "https://docs.example.com/errors/overheated",
    Data =
    {
        { "TimeStamp", DateTime.UtcNow },
        { "Cause", "Exceeded maximum RPM for 30+ seconds" },
        { "PetName", PetName }
    }
};
```

### Reading Data in a Handler

```csharp
catch (Exception ex)
{
    Console.WriteLine(ex.Message);

    foreach (DictionaryEntry entry in ex.Data)
    {
        Console.WriteLine($"  {entry.Key}: {entry.Value}");
    }
}
```

### Tips & Best Practices

- **Use `Data` to carry context that isn't part of the message.** Things like entity IDs, user IDs, timestamps, or input values that help reproduce the problem.
- **Use string keys consistently.** Adopt a naming convention (e.g., `"OrderId"`, not `"order_id"` or `"oid"`) so logging frameworks can index them reliably.
- **Don't store sensitive data.** Passwords, tokens, or PII in `Data` can leak into logs. Treat it like a log field.
- **Don't store large or complex objects.** `Data` values should be simple types (strings, numbers, dates). Large objects bloat serialization and can cause secondary exceptions.
- **Enrich at each layer, don't replace.** As an exception bubbles up, each catch block can add its own context without losing what lower layers contributed.
- **Prefer `DateTime.UtcNow` over `DateTime.Now`** for timestamps to avoid timezone confusion in distributed systems.
- **Use `throw;` (not `throw ex;`)** after adding data so you preserve the original stack trace.
