---
tags: [csharp, asp-net-core, hosting, kestrel]
---


> [!tip] Complete Summary
> The ASP.NET Core hosting model is built around **Kestrel**, a cross-platform, high-performance web server embedded directly in the application. Kestrel can run as an edge server or behind a reverse proxy like **Nginx**, **IIS**, or **Apache** -- with the reverse proxy pattern being the recommended approach for production internet-facing workloads.
>
> When deploying to **IIS on Windows**, you choose between **in-process hosting** (faster, runs inside `w3wp.exe`) and **out-of-process hosting** (better isolation, runs Kestrel in a separate process). In-process is the default starting from .NET 6.
>
> For Windows-specific scenarios requiring **Windows Authentication** or **port sharing**, **HTTP.sys** provides a kernel-mode alternative to Kestrel -- though it sacrifices cross-platform compatibility.
>
> **Self-hosted deployments** use `dotnet publish` to produce framework-dependent or self-contained output, which can run as a **Windows Service** or a **Linux systemd daemon**. **Docker** is the dominant deployment model in cloud environments, using multi-stage builds with Microsoft's official `sdk` and `aspnet` images.
>
> **Kestrel configuration** covers endpoints, TLS certificates, connection limits, and protocol versions (HTTP/1.1, HTTP/2, HTTP/3). **Port configuration** follows a clear precedence: code overrides CLI arguments, which override environment variables, which override `appsettings.json`.
>
> A typical production setup places Kestrel behind Nginx with TLS termination at the proxy, forwarded headers middleware in the app, and the application listening only on the loopback interface. This architecture cleanly separates concerns and allows independent scaling and patching of each component.
