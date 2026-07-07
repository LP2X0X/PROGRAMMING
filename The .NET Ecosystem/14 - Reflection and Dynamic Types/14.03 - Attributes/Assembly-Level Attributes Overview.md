---
tags:
 - csharp
 - attributes
 - assemblies
---

## Assembly-Level Attributes

Attributes can target the entire assembly using the `[assembly:]` prefix. These appear outside any namespace or type definition.

```csharp
using System.Runtime.CompilerServices;
using System.Reflection;

[assembly: InternalsVisibleTo("MyApp.Tests")]
[assembly: AssemblyVersion("2.1.0.0")]
[assembly: AssemblyCompany("My Company")]
```

---

## Where to Put Them

Place assembly-level attributes in any `.cs` file — by convention a dedicated file:

| .NET Version        | Convention                                                        |
| -------------------- | ----------------------------------------------------------------- |
| .NET Framework       | `AssemblyInfo.cs` (in `Properties` folder)                        |
| .NET Core / .NET 5+  | Any file — commonly `GlobalUsings.cs` or a custom `AssemblyAttributes.cs` |

```ad-note
In modern .NET, most assembly-level attributes (Version, Company, Product, etc.) are auto-generated from `.csproj` properties. The `AssemblyInfo.cs` file is auto-generated and should **not** be manually edited. Only attributes like `[InternalsVisibleTo]` need to be placed manually.
```

---

## Common Assembly-Level Attributes

| Attribute                | Purpose                        | Modern .NET equivalent in `.csproj` |
| ------------------------ | ------------------------------ | ----------------------------------- |
| `[AssemblyVersion]`      | Assembly version               | `<AssemblyVersion>`                 |
| `[AssemblyFileVersion]`  | File version (Windows properties) | `<FileVersion>`                  |
| `[AssemblyTitle]`        | Assembly title                 | `<Title>`                           |
| `[AssemblyCompany]`      | Company name                   | `<Company>`                         |
| `[AssemblyProduct]`      | Product name                   | `<Product>`                         |
| `[AssemblyDescription]`  | Description                    | `<Description>`                     |
| `[InternalsVisibleTo]`   | Expose internal types to friend assembly | No `.csproj` equivalent — must use attribute |

---

## `[InternalsVisibleTo]` — The Most Common Manual Assembly Attribute

This is the most practically important assembly-level attribute in modern .NET. It lets a test project access `internal` types:

```csharp
// In your main project
[assembly: InternalsVisibleTo("MyApp.Tests")]
```

Now the test project can access `internal` classes, methods, and properties without making them `public`.

---

## See Also

- [[Attribute Overview]]
- [[The Role of .NET Assemblies]]
