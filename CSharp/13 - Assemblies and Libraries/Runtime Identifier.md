---
tags:
  - csharp
  - assemblies
  - deployment
---

## What Is a Runtime Identifier?

A **Runtime Identifier (RID)** is a short string that identifies the target platform where a .NET application runs. The .NET build system and NuGet use RIDs to select **platform-specific assets** — native libraries, runtime components, and pre-compiled binaries — from NuGet packages.

RIDs follow this format:

```
[os].[version]-[architecture]-[additional qualifiers]
```

Examples: `win-x64`, `linux-arm64`, `osx-arm64`, `win10-x64`.

```ad-info
You rarely need version-specific RIDs (like `win10-x64`) in modern .NET. The **portable** RIDs (`win-x64`, `linux-x64`, `osx-arm64`) cover the vast majority of deployment scenarios and are recommended by Microsoft.
```

---

## Common RIDs

| RID            | Platform                    |
| -------------- | --------------------------- |
| `win-x64`      | Windows 64-bit (Intel/AMD)  |
| `win-x86`      | Windows 32-bit              |
| `win-arm64`    | Windows ARM64               |
| `linux-x64`    | Linux 64-bit (Intel/AMD)    |
| `linux-arm`    | Linux ARM 32-bit            |
| `linux-arm64`  | Linux ARM64 (e.g., Raspberry Pi 4 64-bit, AWS Graviton) |
| `linux-musl-x64` | Alpine Linux 64-bit (musl libc — common in Docker)   |
| `osx-x64`      | macOS Intel                 |
| `osx-arm64`    | macOS Apple Silicon (M1+)   |

```ad-note
The `linux-musl-*` RIDs exist because Alpine Linux uses **musl** instead of **glibc**. If you deploy to Alpine-based Docker images (which are popular for their small size), you need a musl RID or your native dependencies will fail to load at runtime.
```

---

## Setting RID in the Project File

Specify the RID in your `.csproj` to target a specific platform at build time:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>

    <!-- Single target platform -->
    <RuntimeIdentifier>win-x64</RuntimeIdentifier>

    <!-- Or multiple targets (semicolon-separated) for multi-RID publishing -->
    <!-- <RuntimeIdentifiers>win-x64;linux-x64;osx-arm64</RuntimeIdentifiers> -->
  </PropertyGroup>
</Project>
```

You can also pass the RID on the command line without modifying the project file:

```bash
dotnet publish -r win-x64
dotnet publish -r linux-arm64
```

```ad-warning
`RuntimeIdentifier` (singular) sets a single RID for both build and publish. `RuntimeIdentifiers` (plural) defines a **list** of valid targets but does not select one — you still pass `-r` at publish time to choose. Confusing the two is a common source of build errors.
```

---

## Self-Contained vs Framework-Dependent Deployment

The RID's role differs depending on your deployment mode:

| Aspect                  | Framework-Dependent               | Self-Contained                        |
| ----------------------- | --------------------------------- | ------------------------------------- |
| RID required?           | Optional (used for native assets) | **Required**                          |
| .NET runtime included?  | No — host must have it installed  | Yes — bundled in output               |
| Output size             | Small (your code only)            | Large (60-100+ MB with runtime)       |
| Portability             | Runs on any OS with matching runtime | Runs only on the specified RID      |
| Typical use case        | Server apps, internal tools       | Desktop apps, edge deployments, Docker |

Set the deployment mode in `.csproj`:

```xml
<PropertyGroup>
    <SelfContained>true</SelfContained>      <!-- or false for framework-dependent -->
    <RuntimeIdentifier>linux-x64</RuntimeIdentifier>
    <PublishSingleFile>true</PublishSingleFile>  <!-- optional: bundle into one file -->
</PropertyGroup>
```

---

## The RID Graph

RIDs are organized into a **fallback hierarchy** called the RID graph. When the exact RID is not found in a NuGet package, .NET walks up the graph to find the closest match:

```
win10-x64  -->  win-x64  -->  win  -->  any
```

This means a package that only provides `win-x64` assets will still work on `win10-x64` or `win11-x64` — the graph handles the fallback automatically.

```ad-info
Starting in .NET 8, Microsoft simplified the RID graph significantly by removing most version-specific RIDs. The portable RIDs (`win-x64`, `linux-x64`, etc.) are now the primary targets. Version-specific RIDs like `win10-x64` still work but fall back to the portable form almost immediately.
```

**Reference:** [.NET RID Catalog — Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/core/rid-catalog)

---

## See Also

- [[The Role of .NET Assemblies]]
- [[Format of a .NET Assembly]]
