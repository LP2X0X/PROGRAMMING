---
tags:
  - csharp
  - configuration
---

## Overview

Two ways to map a configuration section to a C# object without the full Options pattern DI registration:

- **`Get<T>()`** — creates a new instance and populates it
- **`Bind()`** — populates an existing instance in place

Both live on `IConfigurationSection`.

---

## Get<T>() — Create a New Instance

```csharp
var settings = builder.Configuration
    .GetSection("SmtpSettings")
    .Get<SmtpSettings>(); // returns SmtpSettings? (nullable)
```

- Creates a **new** `SmtpSettings` object internally
- Populates properties from the matching JSON keys
- Returns `null` if the section does not exist

```ad-warning
title: Null Return
`Get<T>()` returns `null` when the section is missing — always null-check or use the null-forgiving operator deliberately.
```

---

## Bind() — Populate an Existing Instance

```csharp
var settings = new SmtpSettings { Port = 25 }; // default values
builder.Configuration
    .GetSection("SmtpSettings")
    .Bind(settings); // overwrites Port if section has "Port"
```

- Uses an **already-created** object
- Overwrites matching properties; leaves unmatched properties untouched
- Returns `void` — mutates in place
- Useful when the object has constructor logic or defaults you want to preserve

---

## Comparison

| | `Get<T>()` | `Bind()` |
|---|---|---|
| Creates new instance | Yes | No — uses existing |
| Returns | `T?` (the populated object or null) | `void` — mutates in place |
| Best when | Simple one-shot read of config | Object has defaults or constructor logic |

```ad-note
title: Options Pattern vs. Get/Bind
`Get<T>()` and `Bind()` are one-time reads — they produce a snapshot. They do **not** reload when `appsettings.json` changes. For live-reloading, use the full [[Working with Objects in Configuration Files|Options pattern]] with `IOptionsSnapshot<T>` or `IOptionsMonitor<T>`.
```

---

## See Also

- [[Working with Objects in Configuration Files]]
- [[Configuring Applications with Configuration Files]]
