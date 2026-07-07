---
tags: [csharp, asp-net-core, http, status-codes, web-api, fundamentals]
---


**Informational responses** indicate that the server has received the request and is continuing to process it. You will rarely interact with these directly in typical ASP.NET Core development -- they are handled by the HTTP infrastructure (Kestrel, reverse proxies, browsers).

| Code | Name | When It Happens |
|---|---|---|
| **100** | Continue | Client sent headers with `Expect: 100-continue`. Server says "go ahead and send the body." Kestrel handles this automatically. |
| **101** | Switching Protocols | Used during [[WebSockets]] upgrade. The connection switches from HTTP to the WebSocket protocol. |
| **102** | Processing | Indicates the server is working on a long request (WebDAV). Rarely seen. |

> [!ad-note] You Almost Never Set These Yourself
> In ASP.NET Core, 1xx codes are managed by Kestrel and the HTTP stack. When a client initiates a WebSocket connection, Kestrel sends the `101 Switching Protocols` response automatically. You do not return `StatusCode(100)` from your controllers.

```csharp
// You don't manually return 1xx codes.
// WebSocket upgrade is handled by the framework:
app.UseWebSockets();
app.Map("/ws", async context =>
{
    if (context.WebSockets.IsWebSocketRequest)
    {
        // Framework sends 101 Switching Protocols automatically
        var ws = await context.WebSockets.AcceptWebSocketAsync();
        // ... handle WebSocket communication
    }
    else
    {
        context.Response.StatusCode = 400;
    }
});
```

> [!summary] Section Summary
> - 1xx codes are informational -- the server acknowledges receipt and continues processing
> - 100 Continue and 101 Switching Protocols are the two you might encounter
> - Kestrel and the HTTP infrastructure handle these automatically
> - You almost never set 1xx codes manually in your controllers
