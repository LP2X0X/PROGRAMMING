---
tags: [csharp, asp-net-core, project-structure]
---


ASP.NET Core ships with several project templates accessible through the `dotnet new` CLI command. The two most commonly used templates for web development are `webapp` (Razor Pages) and `webapi` (Web API).

### Creating a Razor Pages Application

```bash
dotnet new webapp -n OrderManagement -o ./OrderManagement
```

This generates a Razor Pages project with the following structure:

```
OrderManagement/
  OrderManagement.csproj
  Program.cs
  appsettings.json
  appsettings.Development.json
  Properties/
    launchSettings.json
  wwwroot/
    css/
    js/
    lib/
    favicon.ico
  Pages/
    Shared/
      _Layout.cshtml
      _Layout.cshtml.css
      _ValidationScriptsPartial.cshtml
    _ViewImports.cshtml
    _ViewStart.cshtml
    Index.cshtml
    Index.cshtml.cs
    Error.cshtml
    Error.cshtml.cs
    Privacy.cshtml
    Privacy.cshtml.cs
```

### Creating a Web API Application

```bash
dotnet new webapi -n InventoryApi -o ./InventoryApi
```

This generates a minimal API or controller-based API project:

```
InventoryApi/
  InventoryApi.csproj
  Program.cs
  appsettings.json
  appsettings.Development.json
  Properties/
    launchSettings.json
  Controllers/          (if using --use-controllers)
    WeatherForecastController.cs
  WeatherForecast.cs    (sample model)
```

> [!tip] Minimal API vs Controllers
> Starting with .NET 6, `dotnet new webapi` generates a minimal API project by default. To get the traditional controller-based template, use `dotnet new webapi --use-controllers`. Minimal APIs define endpoints directly in `Program.cs` without the need for controller classes.

### Creating an MVC Application

```bash
dotnet new mvc -n ProductCatalog -o ./ProductCatalog
```

This generates the full MVC structure with Controllers, Models, and Views folders, plus the same foundational files (`Program.cs`, `appsettings.json`, etc.).

> [!summary] Section Summary
> - `dotnet new webapp` creates a Razor Pages project with a `Pages/` folder
> - `dotnet new webapi` creates an API project (minimal by default, controller-based with `--use-controllers`)
> - `dotnet new mvc` creates a full MVC project with `Controllers/`, `Models/`, and `Views/` folders
> - All templates share the same foundational files: `.csproj`, `Program.cs`, `appsettings.json`, `launchSettings.json`
