---
tags: [csharp, asp-net-core, project-structure]
---


The SDK attribute in the `.csproj` file fundamentally changes what your project can do. Choosing the wrong SDK leads to missing features or unnecessary dependencies.

### Comparison Table

| Feature | `Microsoft.NET.Sdk` | `Microsoft.NET.Sdk.Web` |
|---|---|---|
| Project type | Class libraries, console apps | ASP.NET Core web apps |
| Web host support | Not included | Built-in (`WebApplication`) |
| Static files (`wwwroot`) | Not recognized | Auto-included in publish output |
| Razor compilation | Not included | Included automatically |
| Default launch settings | None | `launchSettings.json` support |
| Implicit packages | Base BCL only | ASP.NET Core shared framework |
| `dotnet publish` behavior | DLL only | Includes web assets, configs |

### Example: Class Library csproj

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

</Project>
```

### Example: Web Application csproj

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

</Project>
```

> [!warning] Common Mistake
> If you create a class library and accidentally use `Microsoft.NET.Sdk.Web`, your library will pull in the entire ASP.NET Core shared framework -- adding unnecessary dependencies and increasing package size. Always use `Microsoft.NET.Sdk` for libraries unless they specifically need web features (like a Razor Class Library, which uses `Microsoft.NET.Sdk.Razor`).

> [!summary] Section Summary
> - `Microsoft.NET.Sdk.Web` is for ASP.NET Core applications; it includes the web shared framework, Razor compilation, and static file support
> - `Microsoft.NET.Sdk` is for class libraries and console apps; it includes only the base BCL
> - Using the wrong SDK causes missing features or bloated dependencies
> - Razor Class Libraries use `Microsoft.NET.Sdk.Razor`, a specialized variant
