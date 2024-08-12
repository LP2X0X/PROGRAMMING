- While it is possible to keep all the information needed by your .NET application in the source code, being able to change certain values at runtime is vitally important in most applications of significance. One of the more common options is to use one or more configuration files shipped (or deployed) along with your application’s executable.
- The .NET Framework relied mostly on XML files named app.config (or web.config for ASP.NET applications) for configuration. While XML-based configuration files can still be used, the most common method for configuring .NET applications is with JavaScript Object Notation (JSON) files.

---

Configuring applications with configuration files in .NET typically involves using configuration files such as `appsettings.json`, `web.config`, or `app.config`. Here’s a guide on how to use these files for configuring a .NET application:

### Using `appsettings.json`

`appsettings.json` is commonly used in .NET Core and .NET 5+ applications for configuration. Here’s how to use it:

1. **Create `appsettings.json` File**

   Create a file named `appsettings.json` in the root of your project.

   ```json
   {
     "Logging": {
       "LogLevel": {
         "Default": "Information",
         "Microsoft": "Warning",
         "Microsoft.Hosting.Lifetime": "Information"
       }
     },
     "ConnectionStrings": {
       "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=aspnet-YourApp;Trusted_Connection=True;MultipleActiveResultSets=true"
     }
   }
   ```

2. **Load Configuration in `Program.cs` or `Startup.cs`**

   Load the configuration in your application startup code.

   ```csharp
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

   ```csharp
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

           // Access configuration values
           var connectionString = Configuration.GetConnectionString("DefaultConnection");
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

### Using `web.config` or `app.config`

For .NET Framework applications, you typically use `web.config` (for web applications) or `app.config` (for desktop applications).

1. **Create `web.config` or `app.config`**

   Example `web.config`:

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <configuration>
     <connectionStrings>
       <add name="DefaultConnection" connectionString="Server=(localdb)\mssqllocaldb;Database=aspnet-YourApp;Trusted_Connection=True;MultipleActiveResultSets=true" providerName="System.Data.SqlClient" />
     </connectionStrings>
     <appSettings>
       <add key="Setting1" value="Value1" />
     </appSettings>
   </configuration>
   ```

2. **Access Configuration in Code**

   ```csharp
   using System.Configuration;

   public class MyApp
   {
       public void MyMethod()
       {
           var connectionString = ConfigurationManager.ConnectionStrings["DefaultConnection"].ConnectionString;
           var setting1 = ConfigurationManager.AppSettings["Setting1"];
       }
   }
   ```

### Summary

- Use `appsettings.json` for .NET Core and .NET 5+ applications.
- Use `web.config` or `app.config` for .NET Framework applications.
- Load the configuration in the startup code and access it via the `IConfiguration` interface in .NET Core/5+ or via `ConfigurationManager` in .NET Framework.

This approach helps in managing configurations in a centralized manner, making it easier to maintain and update configuration settings.