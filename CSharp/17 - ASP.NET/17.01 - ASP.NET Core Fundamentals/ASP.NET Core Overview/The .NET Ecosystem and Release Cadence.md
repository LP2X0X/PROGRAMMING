---
tags: [csharp, asp-net-core, fundamentals, web]
---


### Release Schedule

Microsoft follows a predictable annual release cadence:

- **Even-numbered releases** (.NET 6, 8, 10) are **LTS (Long-Term Support)** -- 3 years of support
- **Odd-numbered releases** (.NET 7, 9) are **STS (Standard-Term Support)** -- 18 months of support

```
.NET 6 (LTS) -- Nov 2021 to Nov 2024
.NET 7 (STS) -- Nov 2022 to May 2024
.NET 8 (LTS) -- Nov 2023 to Nov 2026
.NET 9 (STS) -- Nov 2024 to May 2026
.NET 10 (LTS) -- Nov 2025 to Nov 2028  (projected)
```

> [!tip] Choosing a Version
> For production applications, prefer the latest **LTS** release unless you specifically need a feature from the latest STS release. As of mid-2026, **.NET 8** is the current LTS and **.NET 10** is newly released as the next LTS.

### SDK and Runtime

The .NET ecosystem has two main components:

- **.NET SDK** -- includes the compiler, CLI tools (`dotnet` command), and project templates. Used for development.
- **.NET Runtime** -- the minimal runtime needed to execute a .NET application. Used for deployment.

```bash
# Check installed SDKs and runtimes
dotnet --list-sdks
dotnet --list-runtimes

# Check current SDK version
dotnet --version
```

### The dotnet CLI

The `dotnet` CLI is the primary tool for creating, building, running, and publishing ASP.NET Core apps:

```bash
# Create a new web API project
dotnet new webapi -n InventoryApi

# Restore NuGet packages
dotnet restore

# Build the project
dotnet build

# Run with hot reload
dotnet watch run

# Publish for deployment
dotnet publish -c Release -o ./publish
```

> [!summary] Section Summary
> - .NET follows an annual release cycle: even = LTS (3 years), odd = STS (18 months)
> - Prefer the latest LTS release for production workloads
> - The SDK is for development; the Runtime is the minimal deployment target
> - The `dotnet` CLI handles project creation, building, running, and publishing
