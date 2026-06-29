---
tags: [csharp, asp-net-core, startup, program]
---


The final call in `Program.cs` is `app.Run()`, which starts the web server and blocks the calling thread until shutdown.

```csharp
app.Run();
```

### What app.Run() Does

1. Starts Kestrel (or the configured web server).
2. Begins listening on the configured URLs and ports.
3. Blocks the main thread, keeping the application alive.
4. Listens for shutdown signals (`Ctrl+C`, `SIGTERM`, `app.StopAsync()`).
5. Triggers graceful shutdown: stops accepting new connections, drains existing requests, disposes services.

### Specifying URLs

```csharp
// Via app.Run parameter
app.Run("https://localhost:7001");

// Via builder configuration
builder.WebHost.UseUrls("https://localhost:5001", "http://localhost:5000");

// Via environment variable
// ASPNETCORE_URLS=https://localhost:5001;http://localhost:5000

// Via command-line argument
// dotnet run --urls "https://localhost:5001"
```

### RunAsync for Non-Blocking Scenarios

```csharp
// Useful in integration tests or background service scenarios
await app.RunAsync();
```

> [!tip] Run vs RunAsync vs Start
> - `app.Run()` blocks the calling thread until the app shuts down.
> - `app.RunAsync()` returns a `Task` that completes on shutdown (useful with `await`).
> - `app.Start()` / `app.StartAsync()` starts the server without blocking — you manage the lifetime yourself.

> [!summary] Section Summary
> - `app.Run()` starts Kestrel and blocks until the application shuts down.
> - URLs can be configured via code, environment variables, or command-line arguments.
> - Graceful shutdown drains existing requests before terminating.
> - `RunAsync` and `StartAsync` offer non-blocking alternatives for advanced scenarios.
