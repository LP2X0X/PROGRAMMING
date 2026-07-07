---
tags:
  - csharp
  - asp-net-core
  - configuration
  - options-pattern
---


**`PostConfigure<T>`** runs **after** all `Configure<T>` calls have executed. It is the last chance to set defaults, override values, or apply computed properties.

### Basic PostConfigure

```csharp
builder.Services.PostConfigure<SmtpSettings>(options =>
{
    // Set a default timeout if none was specified in configuration
    if (options.Timeout == TimeSpan.Zero)
        options.Timeout = TimeSpan.FromSeconds(30);

    // Force SSL in production
    if (!builder.Environment.IsDevelopment())
        options.EnableSsl = true;
});
```

### Execution Order

The configuration pipeline runs in this exact order:

1. **`Configure<T>(section)`** -- binds values from `IConfiguration`
2. **`Configure<T>(action)`** -- applies lambda overrides (in registration order)
3. **`PostConfigure<T>(action)`** -- runs after all `Configure` calls
4. **Validation** -- `ValidateDataAnnotations()`, `.Validate()`, `IValidateOptions<T>`

```csharp
// Step 1: Bind from appsettings.json
builder.Services.Configure<SmtpSettings>(
    builder.Configuration.GetSection("Smtp"));

// Step 2: Override in code
builder.Services.Configure<SmtpSettings>(o => o.Port = 587);

// Step 3: PostConfigure -- runs AFTER all Configure calls
builder.Services.PostConfigure<SmtpSettings>(o =>
    o.Timeout = o.Timeout == TimeSpan.Zero
        ? TimeSpan.FromSeconds(30)
        : o.Timeout);

// Step 4: Validation runs last
builder.Services.AddOptions<SmtpSettings>()
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

### Named PostConfigure

PostConfigure works with named options too:

```csharp
// Apply to a specific named option
builder.Services.PostConfigure<SmtpSettings>("Marketing", options =>
{
    options.EnableSsl = true;
});

// Apply to ALL named options (including the default)
builder.Services.PostConfigureAll<SmtpSettings>(options =>
{
    options.Timeout = TimeSpan.FromSeconds(30);
});
```

> [!warning] Common Misconception
> `PostConfigure` does **not** mean "after the app starts." It means "after all `Configure` registrations are applied to the options instance." It still runs at the time the options are first resolved (or at startup if `ValidateOnStart` is set).

> [!summary] Section Summary
> `PostConfigure<T>` runs after all `Configure<T>` calls, allowing you to set defaults and enforce invariants. Use `PostConfigureAll<T>` to apply defaults across all named instances. Validation runs after PostConfigure.
