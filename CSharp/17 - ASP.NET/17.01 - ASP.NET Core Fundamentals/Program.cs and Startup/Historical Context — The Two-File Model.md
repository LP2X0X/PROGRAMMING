---
tags: [csharp, asp-net-core, startup, program]
---


Before .NET 6, ASP.NET Core applications used **two separate files** to configure the application:

1. **Program.cs** — Created and configured the host (Kestrel, logging, configuration sources).
2. **Startup.cs** — Registered services (`ConfigureServices`) and built the middleware pipeline (`Configure`).

### Pre-.NET 6 Program.cs

```csharp
// Program.cs (.NET 5 and earlier)
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

### Pre-.NET 6 Startup.cs

```csharp
// Startup.cs (.NET 5 and earlier)
public class Startup
{
    public IConfiguration Configuration { get; }

    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllersWithViews();
        services.AddDbContext<InventoryContext>(options =>
            options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
        services.AddScoped<IOrderService, OrderService>();
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
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

        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllerRoute(
                name: "default",
                pattern: "{controller=Home}/{action=Index}/{id?}");
        });
    }
}
```

> [!ad-note] Why Two Files?
> The two-file model enforced separation of concerns: `Program.cs` dealt with **hosting** (how the app runs) while `Startup.cs` dealt with **application logic** (what services exist and how requests flow). This was a clean design, but it introduced ceremony and indirection that made simple apps harder to set up.

> [!summary] Section Summary
> - Pre-.NET 6 apps used `Program.cs` (host creation) and `Startup.cs` (services + pipeline).
> - `Startup.ConfigureServices` registered DI services; `Startup.Configure` built the middleware pipeline.
> - The `IHostBuilder` pattern drove host creation through `Host.CreateDefaultBuilder`.
> - This model was clean but verbose for small applications.
