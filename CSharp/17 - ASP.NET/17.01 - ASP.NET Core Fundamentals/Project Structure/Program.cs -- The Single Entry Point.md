---
tags: [csharp, asp-net-core, project-structure]
---


Since .NET 6, the `Startup.cs` file has been eliminated. All application configuration now lives in a single `Program.cs` file. This file is where you configure services (dependency injection) and the HTTP request pipeline (middleware).

> [!ad-note] Historical Context
> In .NET 5 and earlier, configuration was split between `Program.cs` (host building) and `Startup.cs` (service and middleware configuration). The modern approach merges everything into `Program.cs` using top-level statements. See [[Program.cs and Startup]] for a deeper comparison.

### Web API Program.cs

```csharp
var builder = WebApplication.CreateBuilder(args);

// ----- Service Registration (DI Container) -----
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Register application services
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddDbContext<InventoryContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// ----- Middleware Pipeline -----
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

### Razor Pages Program.cs

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddRazorPages();
builder.Services.AddScoped<IProductRepository, ProductRepository>();

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapRazorPages();

app.Run();
```

### The Two Phases of Program.cs

| Phase | Object | Purpose |
|---|---|---|
| Builder phase | `WebApplicationBuilder` | Register services, configure logging, read configuration |
| App phase | `WebApplication` | Configure middleware pipeline, map endpoints, run the app |

> [!warning] Middleware Order Matters
> The order in which you add middleware to the pipeline is critical. For example, `UseAuthentication()` must come before `UseAuthorization()`, and `UseStaticFiles()` should come early so static file requests short-circuit the pipeline. Incorrect ordering is a common source of bugs.

> [!summary] Section Summary
> - `Program.cs` is the single entry point since .NET 6, replacing the old `Program.cs` + `Startup.cs` split
> - The builder phase registers services (DI); the app phase configures middleware
> - Middleware order in the pipeline is significant and a common source of errors
> - See [[Program.cs and Startup]] for detailed coverage of this file
