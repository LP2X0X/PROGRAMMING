---
tags:
 - csharp
 - assemblies
 - localization
---

## What Are Satellite Assemblies?

**Satellite assemblies** are compiled DLLs containing localized resources (strings, images, etc.) for a specific culture. They let a single application support multiple languages by loading the right assembly based on the user's culture settings.

## How They Work

The main assembly contains default-language resources. Each additional language gets its own satellite assembly deployed in a culture-named subdirectory:

```
MyApp/
├── MyApp.exe                    (default resources — English)
├── en-US/
│   └── MyApp.resources.dll      (English US overrides)
├── fr-FR/
│   └── MyApp.resources.dll      (French)
└── ja-JP/
    └── MyApp.resources.dll      (Japanese)
```

The .NET runtime automatically selects the correct satellite assembly based on `CultureInfo.CurrentCulture`.

## Creating Satellite Assemblies

### 1. Create Resource Files (.resx)

**Default** — `Resources.resx`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<root>
  <data name="HelloWorld" xml:space="preserve">
    <value>Hello World</value>
  </data>
</root>
```

**French** — `Resources.fr-FR.resx`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<root>
  <data name="HelloWorld" xml:space="preserve">
    <value>Bonjour le monde</value>
  </data>
</root>
```

### 2. Compile and Deploy

In modern .NET, just add `.resx` files to the project — MSBuild handles compilation and deployment into culture subdirectories automatically.

```ad-note
title: Legacy Tools
The `resgen` (Resource File Generator) and `al` (Assembly Linker) command-line tools were used in .NET Framework projects to manually compile and link satellite assemblies. Modern .NET SDK projects do not require these tools.
```

### 3. Access Resources in Code

```csharp
var rm = new ResourceManager("MyApp.Resources", typeof(Program).Assembly);

CultureInfo.CurrentCulture = new CultureInfo("fr-FR");
Console.WriteLine(rm.GetString("HelloWorld"));  // "Bonjour le monde"

CultureInfo.CurrentCulture = new CultureInfo("en-US");
Console.WriteLine(rm.GetString("HelloWorld"));  // "Hello World"
```

## Resource Fallback Chain

When a culture-specific resource is not found, .NET falls back through a chain:

1. **Specific culture** (`fr-FR`)
2. **Neutral culture** (`fr`)
3. **Default resources** (main assembly)

## Advantages

- Update translations without recompiling the main app
- Add new languages by deploying new satellite assemblies
- Clean separation of concerns — code vs. content

## See Also
- [[The Role of .NET Assemblies]]
