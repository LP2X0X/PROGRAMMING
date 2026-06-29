---
tags: [csharp, asp-net-core, project-structure]
---


The `dotnet new` command supports many templates beyond `webapp` and `webapi`. Here is a reference table of the most commonly used ones:

| Template | Short Name | Description |
|---|---|---|
| ASP.NET Core Web App (Razor Pages) | `webapp` | Razor Pages with layouts, static files |
| ASP.NET Core Web API | `webapi` | Minimal API or controller-based API |
| ASP.NET Core MVC | `mvc` | Full MVC with Controllers, Views, Models |
| ASP.NET Core Empty | `web` | Bare-bones project with just `Program.cs` |
| Blazor Web App | `blazor` | Blazor with server and/or WebAssembly rendering |
| Worker Service | `worker` | Long-running background service (no HTTP) |
| Class Library | `classlib` | Reusable .NET class library |
| Console Application | `console` | Command-line application |
| ASP.NET Core with React | `react` | ASP.NET Core backend with React SPA frontend |
| ASP.NET Core with Angular | `angular` | ASP.NET Core backend with Angular SPA frontend |
| gRPC Service | `grpc` | gRPC service project |
| Razor Class Library | `razorclasslib` | Reusable Razor components library |
| xUnit Test | `xunit` | Unit test project using xUnit |
| NUnit Test | `nunit` | Unit test project using NUnit |
| Solution File | `sln` | Empty solution file |

### Useful CLI Options

```bash
# List all installed templates
dotnet new list

# Create with specific .NET version
dotnet new webapi -n MyApi --framework net9.0

# Create without HTTPS configuration
dotnet new webapp -n MyApp --no-https

# Create API with controllers instead of minimal API
dotnet new webapi -n MyApi --use-controllers

# Create in a specific output directory
dotnet new mvc -n ProductCatalog -o ./src/ProductCatalog

# Get help for a specific template
dotnet new webapi --help
```

> [!example] Building a Solution from CLI
> ```bash
> # Create solution and projects
> dotnet new sln -n OrderManagement
> dotnet new webapi -n OrderManagement.Api -o src/OrderManagement.Api --use-controllers
> dotnet new classlib -n OrderManagement.Core -o src/OrderManagement.Core
> dotnet new xunit -n OrderManagement.Tests -o tests/OrderManagement.Tests
>
> # Add projects to solution
> dotnet sln add src/OrderManagement.Api
> dotnet sln add src/OrderManagement.Core
> dotnet sln add tests/OrderManagement.Tests
>
> # Add project references
> dotnet add src/OrderManagement.Api reference src/OrderManagement.Core
> dotnet add tests/OrderManagement.Tests reference src/OrderManagement.Core
> ```

> [!summary] Section Summary
> - `dotnet new list` shows all available templates
> - Common templates include `webapp`, `webapi`, `mvc`, `web`, `blazor`, `worker`, `classlib`, `console`
> - Templates accept options like `--use-controllers`, `--no-https`, `--framework`
> - Use `dotnet new sln` combined with `dotnet sln add` to scaffold multi-project solutions from the CLI
