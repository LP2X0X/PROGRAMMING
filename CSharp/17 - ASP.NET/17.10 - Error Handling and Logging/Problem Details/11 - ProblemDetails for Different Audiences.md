---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - problem-details
  - api
---


A well-designed ProblemDetails response serves two audiences simultaneously:

## Machine Clients (Programs, SDKs)

Machine clients use:
- **`type`** -- to identify the error programmatically and branch logic accordingly
- **`status`** -- to determine the HTTP status code without parsing headers
- **`errors`** (ValidationProblemDetails) -- to map field-level errors to form fields
- **Custom extensions** (e.g., `errorCode`) -- for specific business logic branching

```csharp
// Client-side code that handles ProblemDetails
var response = await httpClient.PostAsJsonAsync("/api/orders", order);

if (!response.IsSuccessStatusCode)
{
    var problem = await response.Content
        .ReadFromJsonAsync<ProblemDetails>();

    switch (problem?.Type)
    {
        case "https://example.com/errors/insufficient-funds":
            var balance = problem.Extensions["currentBalance"];
            ShowInsufficientFundsDialog(balance);
            break;

        case "https://example.com/errors/product-unavailable":
            ShowProductUnavailableMessage();
            break;

        default:
            ShowGenericError(problem?.Detail ?? "Unknown error");
            break;
    }
}
```

## Human Users (Developers, Support Staff)

Human readers use:
- **`title`** -- a quick summary of what went wrong
- **`detail`** -- the specific explanation for this occurrence
- **`instance`** -- which request caused the error
- **`traceId`** -- to find the full details in server logs

## Design Guidelines

```csharp
// GOOD: Serves both audiences
new ProblemDetails
{
    // Machine-readable identifier
    Type = "https://example.com/errors/order-limit-exceeded",

    // Same for every occurrence of this problem type
    Title = "Order Limit Exceeded",

    Status = 422,

    // Specific to this occurrence -- human-readable
    Detail = "Customer 'Acme Corp' has reached the monthly order limit " +
             "of 1000 orders. Current count: 1000. " +
             "Limit resets on 2026-07-01.",

    // The request that caused it
    Instance = "/api/orders",

    Extensions =
    {
        // Machine-readable extension for programmatic handling
        ["errorCode"] = "ORDER_LIMIT_EXCEEDED",
        ["currentCount"] = 1000,
        ["limit"] = 1000,
        ["resetsAt"] = "2026-07-01T00:00:00Z",
        ["traceId"] = httpContext.TraceIdentifier
    }
};
```

> [!warning] Common Misconception
> Do not conflate `title` and `detail`. The `title` is a **generic description of the problem type** -- it should be the same for every occurrence (like "Not Found" or "Validation Failed"). The `detail` is **specific to this occurrence** -- it should describe exactly what went wrong with this particular request. Putting instance-specific information in `title` defeats the purpose of having both fields.

> [!summary] Section Summary
> - ProblemDetails serves two audiences: machines (via `type`, `status`, extensions) and humans (via `title`, `detail`, `instance`)
> - `type` is the primary machine-readable identifier for branching client logic
> - `title` is generic to the problem type; `detail` is specific to the occurrence
> - Client SDKs can switch on `type` URI to handle different error scenarios programmatically
> - Include enough context in extensions for clients to react without needing additional API calls
