---
tags: [csharp, asp-net-core, hosting, kestrel]
---


When deploying to IIS on Windows, ASP.NET Core supports two distinct hosting models.

### In-Process Hosting

The ASP.NET Core app runs **inside the IIS worker process** (`w3wp.exe`). IIS handles the HTTP connection, and the request is passed directly to the app through an in-process handler -- no network hop, no socket communication.

```xml
<!-- In web.config -->
<aspNetCore processPath="dotnet"
            arguments=".\OrderService.dll"
            stdoutLogEnabled="false"
            hostingModel="InProcess" />
```

### Out-of-Process Hosting

The ASP.NET Core app runs in a **separate process** alongside Kestrel. IIS acts as a reverse proxy, forwarding requests to Kestrel over a local socket or named pipe.

```xml
<!-- In web.config -->
<aspNetCore processPath="dotnet"
            arguments=".\OrderService.dll"
            stdoutLogEnabled="false"
            hostingModel="OutOfProcess" />
```

### Comparison Table

| Feature | In-Process | Out-of-Process |
|---|---|---|
| Hosting model value | `InProcess` | `OutOfProcess` |
| Process | `w3wp.exe` | `dotnet.exe` + `w3wp.exe` |
| Web server used | IIS HTTP Server | Kestrel (behind IIS as proxy) |
| Performance | Faster (no inter-process hop) | Slightly slower (proxy overhead) |
| Process management | IIS manages lifecycle | IIS starts/monitors the dotnet process |
| App isolation | Shares process with IIS | Separate process provides isolation |
| Cross-platform | Windows + IIS only | Can also run on Linux (without IIS) |
| Windows Auth | Native support | Supported via IIS proxy |
| Default in .NET 6+ | Yes | Must be explicitly configured |

> [!tip] When to Use Each
> **In-Process** is the default and recommended for most IIS deployments due to better performance. Choose **Out-of-Process** when you need process isolation (for example, if your app has memory leaks you want to contain) or when you want the same deployment model across IIS and Linux.

### Verifying the Hosting Model

You can check which model is active at runtime:

```csharp
app.MapGet("/api/hosting-info", (IWebHostEnvironment env) =>
{
    var server = app.Services.GetRequiredService<IServer>();
    return Results.Ok(new
    {
        ServerType = server.GetType().Name,
        Environment = env.EnvironmentName
    });
});
```

- In-process returns `IISHttpServer`
- Out-of-process returns `KestrelServer`

> [!summary] Section Summary
> - In-process hosting runs inside `w3wp.exe` with no inter-process communication overhead
> - Out-of-process hosting runs Kestrel in a separate `dotnet.exe` process with IIS as a reverse proxy
> - In-process is the default and faster; out-of-process provides better isolation
> - The `hostingModel` attribute in `web.config` controls which model is used
