---
tags: [csharp, asp-net-core, fundamentals, web]
---


### Creating the Project

```bash
# Create a new Web API project
dotnet new webapi -n OrderApi --use-controllers

# Navigate to the project
cd OrderApi
```

This generates the following structure (see [[Project Structure]] for a detailed breakdown):

```
OrderApi/
  Properties/
    launchSettings.json      -- Development launch profiles
  Controllers/
    WeatherForecastController.cs
  appsettings.json           -- Default configuration
  appsettings.Development.json -- Environment-specific overrides
  OrderApi.csproj            -- Project file
  Program.cs                 -- Application entry point
```

### Understanding Program.cs

The `Program.cs` file is the entry point. See [[Program.cs and Startup]] for a deep dive.

```csharp
var builder = WebApplication.CreateBuilder(args);

// === Service Registration Phase ===
// Add services to the DI container
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// === Middleware Pipeline Phase ===
// Configure the HTTP request pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

> [!ad-note] Two Phases
> Every ASP.NET Core app has two distinct phases:
> 1. **Service registration** (`builder.Services.Add...`) -- configure DI container
> 2. **Middleware pipeline** (`app.Use...`, `app.Map...`) -- configure HTTP request processing
>
> This separation is important. Services are registered before `builder.Build()` is called. Middleware is configured after.

### Running the Application

```bash
# Run with default settings
dotnet run

# Run with hot reload for development
dotnet watch run

# Run in a specific environment
dotnet run --environment Development
```

The app starts on `https://localhost:5001` and `http://localhost:5000` by default. See [[Environments]] for how environment-specific configuration works.

> [!summary] Section Summary
> - Use `dotnet new webapi` to scaffold a new API project
> - `Program.cs` has two phases: service registration and middleware pipeline configuration
> - Services are added before `builder.Build()`; middleware is configured after
> - `dotnet watch run` enables hot reload during development
> - Default URLs are `https://localhost:5001` and `http://localhost:5000`
