---
tags: [csharp, asp-net-core, project-structure]
---


Every ASP.NET Core project -- regardless of template -- shares a common set of foundational files. This section examines each one in detail.

The `.csproj` file is the MSBuild project file that defines everything about how your application is built, what SDK it uses, which framework it targets, and what NuGet packages it depends on.

### Basic Structure

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <RootNamespace>OrderManagement</RootNamespace>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.0" />
    <PackageReference Include="Serilog.AspNetCore" Version="8.0.0" />
  </ItemGroup>

</Project>
```

### Key Elements

| Element | Purpose |
|---|---|
| `Sdk` | Defines the SDK type; `Microsoft.NET.Sdk.Web` for web projects |
| `TargetFramework` | The .NET version to compile against (e.g., `net8.0`, `net9.0`) |
| `Nullable` | Enables nullable reference types when set to `enable` |
| `ImplicitUsings` | Automatically includes common `using` directives |
| `RootNamespace` | The default namespace for files in the project |
| `PackageReference` | NuGet package dependencies with version pinning |

> [!warning] TargetFramework vs TargetFrameworks
> Use `TargetFramework` (singular) for web projects -- ASP.NET Core apps target a single framework. Multi-targeting with `TargetFrameworks` (plural) is used for class libraries that need to support multiple .NET versions.

### Adding Package References via CLI

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer --version 8.0.0
dotnet add package Serilog.AspNetCore
```

This automatically updates the `<ItemGroup>` section in your `.csproj` file.

> [!summary] Section Summary
> - The `.csproj` file controls SDK, target framework, nullable settings, and package references
> - `Microsoft.NET.Sdk.Web` is the SDK for all ASP.NET Core web projects
> - Package references are managed via `<PackageReference>` elements or the `dotnet add package` CLI command
> - `ImplicitUsings` reduces boilerplate by auto-including common namespaces
