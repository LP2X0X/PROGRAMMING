---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - problem-details
  - api
---


The ProblemDetails standard allows custom extension members beyond the five standard fields. These appear as additional JSON properties at the top level of the response.

```json
{
  "type": "https://example.com/errors/insufficient-funds",
  "title": "Insufficient Funds",
  "status": 422,
  "detail": "Account balance is $50.00 but the transaction requires $75.00.",
  "instance": "/api/transactions/789",
  "traceId": "00-abc123...",
  "errorCode": "INSUFFICIENT_FUNDS",
  "currentBalance": 50.00,
  "requiredAmount": 75.00,
  "supportUrl": "https://support.example.com/billing"
}
```

## Adding Extensions in Code

```csharp
var problemDetails = new ProblemDetails
{
    Type = "https://example.com/errors/insufficient-funds",
    Title = "Insufficient Funds",
    Status = 422,
    Detail = $"Account balance is {balance:C} but the transaction requires {amount:C}."
};

// Use the Extensions dictionary
problemDetails.Extensions["errorCode"] = "INSUFFICIENT_FUNDS";
problemDetails.Extensions["currentBalance"] = balance;
problemDetails.Extensions["requiredAmount"] = amount;
problemDetails.Extensions["supportUrl"] = "https://support.example.com/billing";

return new ObjectResult(problemDetails) { StatusCode = 422 };
```

## Common Extension Members

| Extension | Purpose | Example Value |
|---|---|---|
| `traceId` | Correlate with server logs | `"00-84f1c85e..."` |
| `errorCode` | Machine-readable error identifier | `"PRODUCT_NOT_FOUND"` |
| `timestamp` | When the error occurred | `"2026-06-18T14:30:00Z"` |
| `helpUrl` | Link to documentation about the error | `"https://docs.example.com/errors/auth"` |
| `retryAfter` | When to retry (for rate limiting) | `60` (seconds) |
| `validationErrors` | Detailed validation context | `[{ "field": "email", "rule": "format" }]` |

> [!tip]
> Choose **stable, documented** extension member names and keep them consistent across your entire API. Clients will build logic around these names. Treat extension names as part of your API contract -- changing them is a breaking change. Document your custom extensions in your API reference alongside the standard ProblemDetails members.

> [!ad-note]
> Extensions are serialized as top-level properties in the JSON response due to the `[JsonExtensionData]` attribute on the `Extensions` dictionary. They do not appear inside a nested `"extensions"` object -- they sit alongside `type`, `title`, `status`, etc.

> [!summary] Section Summary
> - ProblemDetails supports custom extension members via the `Extensions` dictionary
> - Extensions appear as top-level JSON properties alongside the standard members
> - Common extensions: `errorCode`, `traceId`, `timestamp`, `helpUrl`, `retryAfter`
> - Treat extension names as part of your API contract -- keep them stable and documented
