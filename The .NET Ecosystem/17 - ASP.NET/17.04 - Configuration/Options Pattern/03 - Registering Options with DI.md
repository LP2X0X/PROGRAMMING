---
tags:
  - csharp
  - asp-net-core
  - configuration
  - options-pattern
---


You register Options in `Program.cs` (or `Startup.ConfigureServices` in older projects) using the `Configure<T>` extension method on `IServiceCollection`.

### Basic Registration

```csharp
var builder = WebApplication.CreateBuilder(args);

// Bind the "Smtp" section to SmtpSettings
builder.Services.Configure<SmtpSettings>(
    builder.Configuration.GetSection(SmtpSettings.SectionName));
```

### What Happens Under the Hood

When you call `services.Configure<SmtpSettings>(section)`:

1. The framework registers `IOptions<SmtpSettings>`, `IOptionsSnapshot<SmtpSettings>`, and `IOptionsMonitor<SmtpSettings>` in the DI container
2. When these interfaces are resolved, the framework reads the configuration section and binds it to a new `SmtpSettings` instance
3. All three interfaces are available from a single `Configure<T>` call

Each call to `Configure<T>` sets up the following series of actions internally:

1. Creates an instance of `ConfigureOptions<T>`, which indicates that `IOptions<T>` should be configured based on configuration. If `Configure<T>` is called multiple times, multiple `ConfigureOptions<T>` objects will be used, all of which can be applied to create the final object — in much the same way that `IConfiguration` is built from multiple layers.
2. Each `ConfigureOptions<T>` instance binds a section of `IConfiguration` to an instance of the `T` POCO class, setting any public properties on the options class based on the keys in the provided `ConfigurationSection`.
3. The `IOptions<T>` interface is registered in the DI container as a **singleton**, with the final bound POCO object in the `Value` property.

> [!note]
> The section name can have any value — it does not have to match the name of your options class. The binding is based on the key-property name matching within the section, not the section name itself.

### Alternative: Bind and Get Immediately

Sometimes you need the options value in `Program.cs` itself (before the DI container is built):

```csharp
var smtpSection = builder.Configuration.GetSection(SmtpSettings.SectionName);
var smtpSettings = smtpSection.Get<SmtpSettings>();

// smtpSettings is a plain SmtpSettings object -- not wrapped in IOptions<T>
Console.WriteLine(smtpSettings?.Host);
```

> [!warning] Common Misconception
> `GetSection()` does **not** throw if the section is missing -- it returns an empty `IConfigurationSection`. Your Options class will be created with all default values. Use [[Options Validation]] to catch missing sections at startup.

### Alternative: Bind Method

```csharp
var smtpSettings = new SmtpSettings();
builder.Configuration.GetSection(SmtpSettings.SectionName).Bind(smtpSettings);
```

### Registration with a Lambda (No Config Section)

You can also configure options purely in code:

```csharp
builder.Services.Configure<SmtpSettings>(options =>
{
    options.Host = "smtp.hardcoded.com";
    options.Port = 465;
    options.EnableSsl = true;
});
```

> [!tip] Pro Tip
> You can combine both approaches. The lambda runs **after** the configuration section binding, so you can override specific values:
> ```csharp
> builder.Services.Configure<SmtpSettings>(
>     builder.Configuration.GetSection("Smtp"));
> builder.Services.PostConfigure<SmtpSettings>(options =>
> {
>     // Override or set defaults after binding
>     options.Timeout = TimeSpan.FromSeconds(60);
> });
> ```

> [!summary] Section Summary
> Call `services.Configure<T>(config.GetSection("SectionName"))` to register options. This automatically registers `IOptions<T>`, `IOptionsSnapshot<T>`, and `IOptionsMonitor<T>`. Use `Get<T>()` or `Bind()` for one-off reads outside DI.
