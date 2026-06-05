---
tags:
 - csharp
 - assemblies
---

## What Is an Assembly?

A **.NET assembly** is the fundamental unit of deployment, versioning, and reuse in .NET. Physically, an assembly is either a `*.dll` (class library) or a `*.exe` (executable) file produced by the .NET compiler. But an assembly is much more than a binary file on disk -- it is the building block around which the entire .NET runtime model is organized.

Every piece of managed code you write lives inside an assembly. Every type you define is scoped to an assembly. Every dependency your application needs is expressed as a reference to another assembly.

```ad-note
For the internal structure of an assembly -- the OS header, CLR header, CIL code, type metadata, and manifest -- see [[Format of a .NET Assembly]].
```

Assemblies serve four fundamental roles in the .NET ecosystem:

1. They **promote code reuse** across languages
2. They **establish a type boundary** that uniquely identifies types
3. They are **versionable units** that support side-by-side execution
4. They are **self-describing** through embedded metadata

Each of these roles is explored in detail below.

---

## Assemblies Promote Code Reuse

The most immediately practical role of an assembly is enabling **code reuse**. A **class library** is a `*.dll` containing types that are designed to be consumed by other applications. When you add a project reference or NuGet package to your solution, you are pulling in an assembly to reuse its types.

```ad-note
While class libraries are almost always packaged as `*.dll` files, it is technically possible for an executable (`*.exe`) to expose types that other applications reference. In practice, this is uncommon, but it demonstrates that the distinction between "library" and "executable" is about intent, not a hard technical boundary.
```

What makes .NET's code reuse particularly powerful is that it is **language-independent**. The CLR's type system (**Common Type System**) and language interoperability rules (**Common Language Specification**) mean that:

- A class library written in **C#** can be consumed directly from **VB.NET**, **F#**, or any other .NET language
- You can **derive from a base class** defined in one language and extend it in another
- You can **implement an interface** defined in F# using a C# struct
- You can **catch exceptions** thrown by a VB.NET library in C# code

This works because all .NET languages compile down to **CIL** (Common Intermediate Language) inside their assemblies. At the assembly level, there is no concept of "C# code" or "F# code" -- there is only CIL and metadata. The runtime does not know or care which language produced the assembly.

```ad-important
This language-neutral code reuse is one of the defining features that separates .NET from platforms where reuse is tied to a single language. When you break a monolithic application into multiple assemblies, each assembly can be authored in whichever .NET language is best suited to its purpose.
```

See also: [[CIL Code, Type Metadata, and the Assembly Manifest]] for details on how CIL enables this cross-language interoperability.

---

## Assemblies Establish a Type Boundary

Every type in .NET has a [[Fully Qualified Name]] -- the combination of its **namespace** and its **type name**. For example, the fully qualified name of the `Console` class is `System.Console`.

However, the fully qualified name alone is *not* enough to uniquely identify a type. The **assembly in which a type resides** is also part of its identity. This means two types with the exact same fully qualified name can coexist in a .NET application as long as they live in different assemblies.

Consider this scenario:

| Assembly | Namespace | Type Name | Fully Qualified Name |
|---|---|---|---|
| `MyCars.dll` | `CarLibrary` | `SportsCar` | `CarLibrary.SportsCar` |
| `YourCars.dll` | `CarLibrary` | `SportsCar` | `CarLibrary.SportsCar` |

Even though both types share the fully qualified name `CarLibrary.SportsCar`, the .NET runtime treats them as **completely distinct types** because they come from different assemblies. The runtime's actual type identity is effectively:

**Assembly + Namespace + Type Name**

```ad-warning
This is why you can encounter ambiguous reference errors at compile time. If your project references two assemblies that both define the same fully qualified type name, the compiler cannot determine which one you mean. You resolve this either by using an **extern alias** or by specifying the assembly explicitly. See [[Aliases]] for more on this.
```

This type boundary mechanism is essential for a large ecosystem. Without it, two independently developed libraries could never safely use the same namespace and type name -- any application referencing both would break. The assembly-level scoping prevents such collisions.

---

## Assemblies Are Versionable Units

Every .NET assembly carries a **four-part version number** with the format:

```
<major>.<minor>.<build>.<revision>
```

If you do not explicitly set a version, the default is `1.0.0.0`.

This version number is embedded in the assembly's **manifest** and becomes part of the assembly's identity. When other assemblies reference yours, they record the specific version they were built against.

The key benefit of assembly versioning is **side-by-side execution**: multiple versions of the same assembly can exist and run on a single machine simultaneously. This solves the classic "DLL Hell" problem that plagued earlier Windows development, where installing a newer version of a shared library could break applications that depended on the older version.

```ad-note
In practice, NuGet and the .NET SDK's dependency resolution system manage assembly versions for you. But understanding that the version number is baked into the assembly's identity -- not just a label -- explains why version conflicts can cause runtime binding failures.
```

A typical version number assignment follows this convention:

| Component | Meaning | Example Change |
|---|---|---|
| **Major** | Breaking changes or major new features | `1.0.0.0` to `2.0.0.0` |
| **Minor** | Backward-compatible feature additions | `1.0.0.0` to `1.1.0.0` |
| **Build** | Build iteration or patch | `1.0.0.0` to `1.0.1.0` |
| **Revision** | Hotfix or incremental revision | `1.0.0.0` to `1.0.0.1` |

### NuGet Package and Target Framework Mismatches

A common build error occurs when you install a NuGet package that **requires a newer .NET version** than your project targets. For example, installing a package built for `net8.0` into a project targeting `net6.0`:

```
error NU1202: Package 'SomeLibrary 5.0.0' is not compatible with 'net6.0'.
Package 'SomeLibrary 5.0.0' supports: net8.0
```

This happens because NuGet packages are compiled against a specific **Target Framework Moniker (TFM)**. If your project's TFM is older, the package's APIs may depend on runtime features that don't exist in your version.

```xml
<!-- Your .csproj -->
<TargetFramework>net6.0</TargetFramework>

<!-- But the package requires net8.0 — build fails -->
```

```ad-warning
**How to fix:**

1. **Upgrade your project's target framework** — update `<TargetFramework>` in your `.csproj` to match or exceed what the package requires. This is the cleanest solution.
2. **Use an older version of the package** — find a version that still supports your TFM. In NuGet Package Manager, uncheck "Latest stable" and browse older versions.
3. **Multi-target your project** — if you need to support multiple frameworks, use `<TargetFrameworks>net6.0;net8.0</TargetFrameworks>` (plural). The package will be used only for the compatible target.

Before installing a package, check its **Dependencies** tab on nuget.org — it lists which TFMs are supported.
```

```ad-note
This is different from **assembly version mismatches** (where the version number doesn't match what was expected at compile time). A TFM mismatch means the package was built for a fundamentally different runtime, not just a different version of the same library.
```

---

## Assemblies Are Self-Describing

One of the most architecturally significant roles of a .NET assembly is that it is **self-describing**. An assembly carries within itself all the metadata needed for the runtime to understand what it contains and what it depends on. This metadata comes in two forms:

### The Assembly Manifest

The **manifest** is a block of metadata that describes the assembly itself. It includes:

- The **assembly name**
- The **version number**
- A list of all **external assemblies** this assembly depends on
- Culture/locale information (if applicable)
- Strong name information (if applicable)

The manifest answers the question: *"What is this assembly, and what does it need to run?"*

See [[CIL Code, Type Metadata, and the Assembly Manifest]] for a deeper look at manifest contents.

### Type Metadata

In addition to the manifest, every assembly contains complete **type metadata** -- a detailed description of every type defined within it. This includes:

- All **member** names and signatures (methods, properties, fields, events)
- **Base classes** and **implemented interfaces**
- **Constructors** and their parameters
- Custom attributes applied to types and members

The type metadata answers the question: *"What types does this assembly contain, and what do they look like?"*

### Why Self-Description Matters

Because assemblies carry all of this information internally, the **.NET runtime does not need to consult the Windows registry** to resolve assembly locations or understand their contents. This is a radical departure from the legacy **COM** programming model, where type information and registration data were stored in the Windows registry.

```ad-important
The elimination of registry dependency has two major consequences:

1. **.NET is cross-platform.** Since there is no reliance on the Windows registry, the same assembly resolution model works on Linux, macOS, and other platforms. The runtime reads the assembly's own metadata regardless of the host OS.

2. **Multiple .NET versions can coexist.** Without a shared registry acting as a single point of contention, different versions of the .NET runtime and different versions of assemblies can live side by side on the same machine without interfering with each other.
```

This self-describing nature is also what makes .NET tooling so powerful. Tools like **ILSpy**, **dotPeek**, and the `ildasm` disassembler can fully inspect any assembly without needing external documentation or header files -- everything they need is embedded in the assembly itself.

---

## Summary

| Role | Key Benefit |
|---|---|
| **Promote Code Reuse** | Types can be shared across any .NET language thanks to CIL and the Common Type System |
| **Establish a Type Boundary** | Assembly identity prevents naming collisions -- same fully qualified name in different assemblies = different types |
| **Versionable Units** | Four-part version numbers enable side-by-side execution and eliminate DLL Hell |
| **Self-Describing** | Embedded manifest and type metadata free .NET from registry dependency, enabling cross-platform support and rich tooling |

These four roles work together to make assemblies the cornerstone of .NET's deployment, interoperability, and versioning model. For the structural details of what an assembly physically contains, see [[Format of a .NET Assembly]].
