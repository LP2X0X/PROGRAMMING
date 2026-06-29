---
tags:
  - csharp
  - asp-net-core
  - configuration
  - fundamentals
---


**`GetSection`** returns an `IConfigurationSection` representing a subtree of the configuration. This is useful for grouping related settings.

```csharp
IConfigurationSection smtpSection = config.GetSection("Smtp");

string? host = smtpSection["Host"];        // No need for "Smtp:Host"
int port = smtpSection.GetValue<int>("Port");
```

### Checking If a Section Exists

`GetSection` never returns `null`. To check whether a section actually has values, use the `Exists()` extension method:

```csharp
IConfigurationSection section = config.GetSection("FeatureFlags");

if (section.Exists())
{
    // Section has values
}
```

### Enumerating Children

```csharp
IConfigurationSection loggingSection = config.GetSection("Logging:LogLevel");

foreach (IConfigurationSection child in loggingSection.GetChildren())
{
    Console.WriteLine($"{child.Key} = {child.Value}");
}
// Output:
// Default = Information
// Microsoft = Warning
// Microsoft.Hosting.Lifetime = Information
```

### Binding a Section to a POCO

Sections can be bound to a plain C# object, which is the foundation for the [[Options Pattern]]:

```csharp
public class SmtpSettings
{
    public string Host { get; set; } = string.Empty;
    public int Port { get; set; }
    public bool UseSsl { get; set; }
}
```

```csharp
var smtpSettings = new SmtpSettings();
config.GetSection("Smtp").Bind(smtpSettings);

// Or the one-liner equivalent:
SmtpSettings smtp = config.GetSection("Smtp").Get<SmtpSettings>()!;
```

> [!warning] Bind Does Not Throw on Missing Keys
> If a property in the POCO has no matching configuration key, it retains its default value. The binding silently ignores missing keys -- it does **not** throw. Validate your objects after binding if certain properties are required.

> [!summary] Section Summary
> `GetSection` provides scoped access to a subtree of configuration. Sections can be enumerated with `GetChildren()`, checked with `Exists()`, and bound to POCOs with `Bind()` or `Get<T>()` for strongly typed access.
