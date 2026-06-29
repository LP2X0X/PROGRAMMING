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
