---
tags:
  - csharp
  - asp-net-core
  - web-api
  - content-negotiation
  - serialization
---


Out of the box, ASP.NET Core ships with ==only a JSON output formatter== using **System.Text.Json** (STJ). This means that regardless of what a client requests in the `Accept` header, the server returns JSON unless additional formatters are registered.

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers(); // Registers the default JSON formatter

var app = builder.Build();
app.MapControllers();
app.Run();
```

With this setup, ASP.NET Core registers two default formatters:

| Formatter | Direction | Media Type |
|---|---|---|
| `SystemTextJsonOutputFormatter` | Output | `application/json` |
| `SystemTextJsonInputFormatter` | Input | `application/json` |

### What Happens When the Client Requests XML?

If a client sends `Accept: application/xml` but no XML formatter is registered, ASP.NET Core does **not** return `406 Not Acceptable` by default. Instead, it ==falls back to JSON==. This is a deliberate design decision to maximize compatibility.

```http
GET /api/products HTTP/1.1
Accept: application/xml

--- Response ---
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

[{"id":1,"name":"Keyboard","price":49.99}]
```

### Enabling 406 Not Acceptable

If you want strict content negotiation that returns `406` when no formatter matches the `Accept` header, configure `MvcOptions`:

```csharp
builder.Services.AddControllers(options =>
{
    options.ReturnHttpNotAcceptable = true; // Return 406 if no formatter matches
});
```

Now if a client requests `application/xml` and no XML formatter is registered:

```http
GET /api/products HTTP/1.1
Accept: application/xml

--- Response ---
HTTP/1.1 406 Not Acceptable
```

> [!tip]
> Setting `ReturnHttpNotAcceptable = true` is recommended for public APIs. It forces clients to request formats you actually support rather than silently receiving JSON when they expected something else.

### Respect Browser Accept Headers

Browsers send complex `Accept` headers like `text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8`. The `*/*` wildcard means "I accept anything," which will always match JSON. You can control this:

```csharp
builder.Services.AddControllers(options =>
{
    options.RespectBrowserAcceptHeader = true; // Don't treat browser requests specially
});
```

> [!summary] Section Summary
> ASP.NET Core defaults to JSON via System.Text.Json and falls back to JSON even when the client requests an unsupported format. Enable `ReturnHttpNotAcceptable = true` to return 406 for unsupported formats. Use `RespectBrowserAcceptHeader` to control how browser `Accept` headers are interpreted.
