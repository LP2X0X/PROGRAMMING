---
tags: [csharp, asp-net-core, startup, program]
---


Starting with .NET 6, the ASP.NET Core team introduced the **minimal hosting model**. This collapses `Program.cs` and `Startup.cs` into a single `Program.cs` file using **top-level statements**.

```csharp
// Program.cs (.NET 6+ minimal hosting model)
var builder = WebApplication.CreateBuilder(args);

// === Builder Phase: Configure Services ===
builder.Services.AddControllersWithViews();
builder.Services.AddDbContext<InventoryContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
builder.Services.AddScoped<IOrderService, OrderService>();

var app = builder.Build();

// === App Phase: Configure Middleware Pipeline ===
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

> [!tip] Key Insight
> The minimal hosting model is not a different framework. It is the **same ASP.NET Core** with a simplified bootstrapping API. Under the hood, `WebApplication.CreateBuilder` still creates an `IHostBuilder` and calls `Host.CreateDefaultBuilder`. The simplification is entirely at the API surface.

> [!summary] Section Summary
> - .NET 6+ merges both files into a single `Program.cs` using top-level statements.
> - The same two-phase pattern exists: builder phase (services) then app phase (pipeline).
> - The underlying infrastructure (`IHostBuilder`, `IHost`) is unchanged.
> - Less ceremony, same power.
