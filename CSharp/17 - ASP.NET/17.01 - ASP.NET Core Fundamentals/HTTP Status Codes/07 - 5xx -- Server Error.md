---
tags:
  - csharp
  - asp-net-core
  - http
  - status-codes
---


**Server error responses** mean ==the problem is on the server's side==. The client's request was valid, but the server failed to process it. These indicate bugs, infrastructure issues, or temporary failures.

| Code | Name | Meaning |
|---|---|---|
| **500** | Internal Server Error | The server encountered an unexpected condition. The catch-all "something broke" code. |
| **502** | Bad Gateway | The server, acting as a gateway or proxy, received an invalid response from an upstream server. |
| **503** | Service Unavailable | The server is temporarily unable to handle requests (overloaded or down for maintenance). |
| **504** | Gateway Timeout | The server, acting as a gateway or proxy, did not receive a timely response from an upstream server. |

## 500 Internal Server Error -- Something Broke

This is the "unhandled exception" of HTTP. In ASP.NET Core, unhandled exceptions are caught by the [[Exception Handling]] middleware and converted to 500 responses.

```csharp
// You should NEVER return 500 manually in normal flow.
// The exception handling middleware does this for you:

// In Program.cs
app.UseExceptionHandler("/error"); // Catches unhandled exceptions → 500

// Or with Problem Details (recommended):
app.UseExceptionHandler();
builder.Services.AddProblemDetails();
```

> [!ad-warning] Never Return 500 for Validation Errors
> A 500 means "the server has a bug." If the client sent bad data, that is a **400** (or 422), not a 500. Returning 500 for validation errors tells the client "this is our fault" when it is actually theirs.
> ```csharp
> // WRONG -- 500 for bad input
> [HttpPost]
> public IActionResult Create(CreateProductDto dto)
> {
>     if (string.IsNullOrEmpty(dto.Name))
>         throw new Exception("Name is required"); // Becomes 500!
>
>     // ...
> }
>
> // CORRECT -- 400 for bad input
> [HttpPost]
> public IActionResult Create(CreateProductDto dto)
> {
>     if (string.IsNullOrEmpty(dto.Name))
>         return BadRequest(new { error = "Name is required" }); // 400
>
>     // ...
> }
> ```

## 502, 503, 504 -- Infrastructure Codes

These are typically not returned by your ASP.NET Core application code directly. They come from reverse proxies (Nginx, IIS, Azure App Gateway, load balancers) or from your code when it acts as a proxy to another service.

```
Client → Nginx (reverse proxy) → Kestrel (your app) → Database / External API
                 │                       │
                 │                       └── If your app crashes → Nginx returns 502
                 │
                 └── If Kestrel doesn't respond in time → Nginx returns 504
```

| Scenario | Who Returns the Code |
|---|---|
| Your app throws an unhandled exception | Your app returns **500** |
| Your app crashes or Kestrel goes down | Reverse proxy (Nginx/IIS) returns **502** |
| Your app is overloaded or starting up | Reverse proxy returns **503** |
| Your app takes too long to respond | Reverse proxy returns **504** |
| Your app calls an external API that fails | Your app might return **502** (acting as a gateway) |

```csharp
// When your app acts as a gateway to another service:
[HttpGet("external-data")]
public async Task<IActionResult> GetExternalData()
{
    try
    {
        var response = await _httpClient.GetAsync("https://api.thirdparty.com/data");

        if (!response.IsSuccessStatusCode)
            return StatusCode(502, new { error = "Upstream service returned an error" });

        var data = await response.Content.ReadFromJsonAsync<ExternalData>();
        return Ok(data);
    }
    catch (TaskCanceledException) // Timeout
    {
        return StatusCode(504, new { error = "Upstream service timed out" });
    }
    catch (HttpRequestException)
    {
        return StatusCode(502, new { error = "Could not reach upstream service" });
    }
}
```

> [!summary] Section Summary
> - 5xx codes mean the server failed to handle a valid request
> - 500: catch-all server error -- unhandled exceptions become 500 via exception handling middleware
> - 502/503/504: infrastructure codes typically returned by reverse proxies, not your application code
> - Never manually return 500 for validation errors -- that is a 400 or 422
> - When your app acts as a gateway to other services, 502 and 504 become relevant
