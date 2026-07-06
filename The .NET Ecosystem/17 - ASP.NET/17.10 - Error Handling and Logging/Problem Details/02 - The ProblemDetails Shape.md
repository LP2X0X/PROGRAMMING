---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - problem-details
  - api
---


The standard defines five core members, all optional:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.5",
  "title": "Not Found",
  "status": 404,
  "detail": "Product with ID 42 was not found.",
  "instance": "/api/products/42"
}
```

## Standard Members

| Member | Type | Purpose | Example |
|---|---|---|---|
| **`type`** | string (URI) | A URI reference that identifies the **problem type**. Ideally points to human-readable documentation. | `"https://example.com/errors/product-not-found"` |
| **`title`** | string | A short, human-readable summary of the problem type. Should be the same for every occurrence of this problem type. | `"Not Found"` |
| **`status`** | integer | The HTTP status code. Included for convenience -- it must match the actual HTTP response status code. | `404` |
| **`detail`** | string | A human-readable explanation specific to **this occurrence** of the problem. Unlike `title`, this changes per request. | `"Product with ID 42 was not found."` |
| **`instance`** | string (URI) | A URI reference that identifies the **specific occurrence** of the problem. Often the request path or a unique error ID. | `"/api/products/42"` |

## ASP.NET Core Additions

ASP.NET Core's `ProblemDetails` class adds a commonly used member:

| Member | Type | Purpose |
|---|---|---|
| **`extensions`** | dictionary | A bag for custom key-value pairs (e.g., `traceId`, `errorCode`, `timestamp`) |

In practice, ASP.NET Core automatically includes a `traceId` extension:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.5",
  "title": "Not Found",
  "status": 404,
  "detail": "Product with ID 42 was not found.",
  "instance": "/api/products/42",
  "traceId": "00-84f1c85e03d87d4cb7eafab94ddf2f58-7f3e26f21c3b254a-00"
}
```

## The ProblemDetails Class in C#

```csharp
// Microsoft.AspNetCore.Mvc.ProblemDetails
public class ProblemDetails
{
    public string? Type { get; set; }
    public string? Title { get; set; }
    public int? Status { get; set; }
    public string? Detail { get; set; }
    public string? Instance { get; set; }

    // Extension members -- any additional key-value pairs
    [JsonExtensionData]
    public IDictionary<string, object?> Extensions { get; set; }
}
```

> [!warning] Common Misconception
> The `type` field is not just a descriptive string -- it is meant to be a **URI** that a client can use to programmatically identify the error type. While the RFC does not require that the URI be dereferenceable (i.e., it does not need to return a web page), it is best practice to point it to documentation that describes the error. Many teams use `about:blank` or an HTTP status reference URI as a default when they have not set up custom error type documentation.

> [!ad-note]
> All members are optional per the RFC. However, you should always include at least `type`, `title`, and `status` for a useful error response. The `detail` field should describe **this specific error occurrence**, not the error type in general.

> [!summary] Section Summary
> - ProblemDetails has five standard members: `type` (URI identifying the problem), `title` (human-readable summary), `status` (HTTP code), `detail` (instance-specific explanation), `instance` (request URI or unique error reference)
> - ASP.NET Core adds an `Extensions` dictionary for custom data like `traceId`
> - The `type` field should be a URI -- ideally pointing to documentation -- not just a descriptive string
> - `title` is the same for every occurrence of a problem type; `detail` is specific to each occurrence
