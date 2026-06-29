---
tags: [csharp, asp-net-core, fundamentals, web]
---


ASP.NET Core ships as a **shared framework** via the `Microsoft.AspNetCore.App` metapackage. This package bundles all the core ASP.NET Core libraries:

- MVC, Razor Pages, Razor runtime compilation
- Kestrel web server
- Authentication and authorization
- SignalR
- gRPC
- Logging, configuration, DI
- Health checks, CORS, response caching

### How It Works

You do **not** need to add an explicit NuGet reference to `Microsoft.AspNetCore.App`. When your project targets `Microsoft.NET.Sdk.Web`, the framework reference is included implicitly:

```xml
<!-- Your .csproj file -->
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
</Project>
```

> [!ad-note] Sdk="Microsoft.NET.Sdk.Web"
> The `Microsoft.NET.Sdk.Web` SDK type is what triggers the implicit framework reference. If you create a class library that needs ASP.NET Core types (e.g., for shared controllers), you must add an explicit framework reference:
> ```xml
> <ItemGroup>
>   <FrameworkReference Include="Microsoft.AspNetCore.App" />
> </ItemGroup>
> ```

### What Is NOT in the Shared Framework

Some packages are **not** included in the shared framework and must be added as separate NuGet packages:

| Package | Purpose |
|---|---|
| `Microsoft.EntityFrameworkCore.*` | Database access (EF Core) |
| `Microsoft.AspNetCore.Authentication.JwtBearer` | JWT token authentication |
| `Microsoft.AspNetCore.Identity.EntityFrameworkCore` | ASP.NET Core Identity with EF |
| `Swashbuckle.AspNetCore` | Swagger/OpenAPI generation |
| `Microsoft.AspNetCore.OpenApi` | Built-in OpenAPI (.NET 9+) |
| `Serilog.AspNetCore` | Structured logging with Serilog |

```xml
<!-- Adding packages not in the shared framework -->
<ItemGroup>
  <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.0" />
  <PackageReference Include="Serilog.AspNetCore" Version="8.0.0" />
</ItemGroup>
```

> [!summary] Section Summary
> - `Microsoft.AspNetCore.App` is the shared framework containing all core ASP.NET Core libraries
> - It is included implicitly when using `Microsoft.NET.Sdk.Web` -- no explicit NuGet reference needed
> - Class libraries that need ASP.NET Core types must add an explicit `<FrameworkReference>`
> - EF Core, JWT auth, Identity, and third-party logging are NOT included and require separate NuGet packages
