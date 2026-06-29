---
tags: [csharp, asp-net-core, hosting, kestrel]
---


ASP.NET Core applications are self-contained executables that do not require an external web server to run. There are several ways to run them directly.

### Running with dotnet CLI

```bash
# Development
dotnet run --project ./src/OrderService/OrderService.csproj

# Production (from published output)
dotnet publish -c Release -o ./publish
dotnet ./publish/OrderService.dll
```

### Framework-Dependent vs Self-Contained

```bash
# Framework-dependent (requires .NET runtime on target machine)
dotnet publish -c Release -o ./publish

# Self-contained (bundles the runtime -- larger but no runtime dependency)
dotnet publish -c Release -r linux-x64 --self-contained -o ./publish

# Single-file deployment
dotnet publish -c Release -r linux-x64 --self-contained -p:PublishSingleFile=true -o ./publish
```

> [!example] Choosing a Deployment Strategy
> **Framework-dependent** is best when your servers already have the .NET runtime installed (common in enterprise environments). **Self-contained** is best for Docker images, edge deployments, or when you cannot control the target environment. **Single-file** is ideal for CLI tools or microservices where you want a single binary to deploy.

### Running as a Windows Service

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Host.UseWindowsService(options =>
{
    options.ServiceName = "OrderProcessingService";
});

var app = builder.Build();

app.MapGet("/api/health", () => Results.Ok("Healthy"));

app.Run();
```

Install and manage with `sc.exe`:

```bash
sc create OrderProcessingService binPath="C:\services\OrderService.exe"
sc start OrderProcessingService
sc stop OrderProcessingService
```

### Running as a Linux systemd Service

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Host.UseSystemd();

var app = builder.Build();
app.Run();
```

Create a unit file at `/etc/systemd/system/orderservice.service`:

```
[Unit]
Description=Order Processing Service
After=network.target

[Service]
Type=notify
ExecStart=/usr/bin/dotnet /opt/orderservice/OrderService.dll
WorkingDirectory=/opt/orderservice
Restart=always
RestartSec=10
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=ASPNETCORE_URLS=http://localhost:5000

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable orderservice
sudo systemctl start orderservice
sudo systemctl status orderservice
```

> [!summary] Section Summary
> - ASP.NET Core apps are self-hosted -- they carry their own web server (Kestrel)
> - `dotnet publish` produces deployable output in framework-dependent or self-contained modes
> - Windows services use `UseWindowsService()`; Linux daemons use `UseSystemd()`
> - Self-contained single-file deployment produces a single executable with no external dependencies
