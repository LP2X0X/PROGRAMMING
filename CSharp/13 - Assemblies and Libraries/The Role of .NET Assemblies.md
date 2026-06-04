---
tags:
  - csharp
  - assemblies
---

## What Is an Assembly?

An **assembly** is the fundamental unit of deployment, versioning, and code reuse in .NET. Physically, it is a `.dll` (class library) or `.exe` (executable) file containing:

- **IL (Intermediate Language) code** — the compiled, platform-neutral bytecode
- **Metadata** — full descriptions of every type, member, interface, and base class
- **A manifest** — a catalog of the assembly's identity, version, culture, external dependencies, and exported types

Every .NET application is composed of one or more assemblies. The runtime loads, verifies, and JIT-compiles them on demand.

```ad-info
Unlike native C/C++ libraries, assemblies are **not** registered in the Windows registry. The runtime resolves them through probing rules, not registry lookups — a deliberate break from the COM model that enables side-by-side versioning and cross-platform deployment.
```

---

## Assemblies Promote Code Reuse

A **code library** (class library) is a `*.dll` containing types intended for consumption by other applications. Although `*.exe` files can also expose types and technically serve as referenced libraries, `*.dll` is the standard packaging.

The reuse that assemblies enable is **language-independent**:

- A library written in C# can be consumed from VB.NET, F#, or any other .NET language without wrappers or translation layers.
- **Cross-language inheritance** — a VB.NET class can derive from a C# base class.
- **Cross-language interface implementation** — an F# interface can be implemented by a C# struct.

This works because all .NET languages compile to the same IL and share the same type system (the [[Common Type System]]). The assembly is the delivery vehicle that makes this interoperability seamless.

---

## Assemblies Establish a Type Boundary

A type's **full identity** in .NET is not just its namespace plus its name — the assembly it lives in is part of the identity too.

> **Identity = Assembly + Namespace + Type Name**

This means two assemblies can define identically-named types without conflict:

```csharp
// In MyCars.dll
namespace CarLibrary
{
    public class SportsCar { /* ... */ }
}

// In YourCars.dll
namespace CarLibrary
{
    public class SportsCar { /* ... */ }
}
```

These are **two completely different types** in the .NET type system, even though their namespace and class name are identical. The assembly boundary disambiguates them. If a project references both, it must use extern aliases or fully assembly-qualified names to resolve the ambiguity.

```ad-note
This is why [[Fully Qualified Name|fully qualified names]] in .NET can include the assembly name, version, culture, and public key token — not just namespace + type name.
```

---

## Assemblies Are Versionable Units

Every assembly carries a four-part version number:

```
<major>.<minor>.<build>.<revision>
```

| Segment    | Typical meaning                        |
| ---------- | -------------------------------------- |
| `major`    | Breaking changes / new major release   |
| `minor`    | Backward-compatible feature additions  |
| `build`    | Build number (often auto-incremented)  |
| `revision` | Patch or hotfix within a build         |

If you do not set a version explicitly, .NET defaults to **1.0.0.0**.

### Setting the Version in `.csproj`

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>

    <!-- Assembly version (used by the runtime for binding) -->
    <AssemblyVersion>2.1.0.0</AssemblyVersion>

    <!-- File version (shown in Windows file properties) -->
    <FileVersion>2.1.3.47</FileVersion>

    <!-- Informational / display version (can include pre-release tags) -->
    <InformationalVersion>2.1.3-beta.1</InformationalVersion>
  </PropertyGroup>
</Project>
```

```ad-warning
`AssemblyVersion` is what the runtime uses for **strong-name binding**. Changing it between releases means older binaries cannot load the new assembly without binding redirects. Many teams keep `AssemblyVersion` stable across minor releases and use `FileVersion` or `InformationalVersion` for human-readable tracking.
```

Multiple versions of the same assembly can coexist on a single machine — in the [[Global Assembly Cache]] (legacy .NET Framework) or via side-by-side deployment in modern .NET.

---

## Assemblies Are Self-Describing

Assemblies carry enough metadata to fully describe themselves without any external registry:

| What the manifest records           | Why it matters                                           |
| ----------------------------------- | -------------------------------------------------------- |
| Assembly name and version           | Runtime identity and version binding                     |
| Culture / locale                    | Enables [[Satellite Assemblies]] for localization        |
| Strong name / public key token      | Tamper detection and unique identity in the GAC          |
| List of referenced assemblies       | Runtime knows every dependency up front                  |
| Exported types                      | Consumers discover available types without source code   |

| What the type metadata records      | Why it matters                                           |
| ----------------------------------- | -------------------------------------------------------- |
| Every class, struct, enum, delegate | Full type catalog — no header files needed               |
| Members (methods, properties, etc.) | IntelliSense, reflection, serialization all work         |
| Base classes and implemented interfaces | Enables runtime type checking and polymorphism        |
| Custom attributes                   | Declarative metadata for frameworks, analyzers, tools    |

Because all of this is embedded in the assembly itself:

- **No Windows registry** is needed to locate or describe assemblies (unlike COM).
- **.NET applications are xcopy-deployable** — copy the files and they run.
- **Cross-platform deployment** works because the metadata travels with the binary, not in an OS-specific store.
- **Multiple .NET versions** can coexist on the same machine without registry conflicts.

```ad-info
You can inspect an assembly's metadata using tools like `ildasm` (IL Disassembler), `dotnet-ildasm`, or the Reflection APIs (`Assembly.GetTypes()`, `Type.GetMembers()`, etc.).
```

---

## See Also

- [[Format of a .NET Assembly]]
- [[Fully Qualified Name]]
- [[Satellite Assemblies]]
- [[Global Assembly Cache]]
- [[Common Type System]]
