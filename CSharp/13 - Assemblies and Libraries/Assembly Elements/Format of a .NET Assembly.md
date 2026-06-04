---
tags:
 - csharp
 - assemblies
---

## Introduction

A .NET assembly (`*.dll` or `*.exe`) is not just a blob of compiled code -- it is a **structured binary file** with multiple distinct layers, each serving a different audience. Some layers exist for the operating system. Some exist for the **CLR** (Common Language Runtime). Some exist for the **JIT compiler**. And some exist for developer tooling like IntelliSense, reflection, and decompilers.

Understanding these layers gives you a mental model for what happens when an assembly is loaded and executed, why certain tools exist, and what information the runtime has access to at any given moment.

Every .NET assembly contains the following elements:

1. An **operating system (PE/COFF) file header** -- read by the OS loader
2. A **CLR file header** -- read by the CLR to identify the file as managed
3. **CIL code** -- the compiled intermediate language, JIT-compiled at runtime
4. **Type metadata** -- a complete description of every type in the assembly
5. An **assembly manifest** -- metadata describing the assembly itself
6. **Optional embedded resources** -- images, strings, configuration, etc.

The diagram below shows how these layers are arranged within the physical file:

```
┌─────────────────────────────────────┐
│   OS (PE/COFF) File Header          │  ← The OS reads this first
├─────────────────────────────────────┤
│   CLR File Header                   │  ← The CLR reads this to load managed code
├─────────────────────────────────────┤
│   CIL Code                          │  ← The JIT compiler converts this to native code
├─────────────────────────────────────┤
│   Type Metadata                     │  ← Reflection, tooling, runtime type resolution
├─────────────────────────────────────┤
│   Assembly Manifest                 │  ← Identity, versioning, dependency references
├─────────────────────────────────────┤
│   Embedded Resources (optional)     │  ← Images, strings, config, icons, etc.
└─────────────────────────────────────┘
```

```ad-note
This note provides a **structural overview** -- enough detail on each element that you understand what it is, why it exists, and how it fits into the whole picture. The sibling notes linked from each section go deeper into individual elements.
```

For the roles that assemblies play in the .NET ecosystem (code reuse, type boundaries, versioning, self-description), see [[The Role of .NET Assemblies]].

---

## The Operating System (Windows) File Header

The outermost layer of a .NET assembly is the **PE (Portable Executable) file header**, also called the **PE/COFF header**. This is *not* .NET-specific -- it is the standard binary format that Windows uses for all executables and DLLs, including native C++ programs, device drivers, and .NET assemblies alike.

The OS loader reads this header *first*, before anything .NET-related happens. Its job is to answer basic questions:

- **Can this file be loaded?** Is it a valid executable?
- **What kind of binary is it?** Console application, GUI application, or DLL?
- **What platform does it target?** x86, x64, ARM?
- **Where are the various sections of the file?** Code, data, resources, etc.

The PE header is composed of several sub-structures:

| Sub-structure | Purpose |
|---|---|
| **DOS Header (MZ stub)** | Legacy compatibility -- begins with the bytes `4D 5A` ("MZ"). Allows the file to print a message like "This program cannot be run in DOS mode" if someone tries to execute it in a DOS environment. |
| **PE Signature** | The bytes `50 45 00 00` ("PE\0\0") that confirm the file is a valid PE image. |
| **COFF Header** | Contains the target machine type, number of sections, and timestamp. |
| **Optional Header** | Despite its name, this is *not* optional for executables. Contains the entry point address, image base, section alignment, and subsystem type (console vs. GUI). |

```ad-note
For .NET assemblies, the Optional Header also contains a **data directory entry** that points to the CLR header. This is the bridge between the OS-level structure and the .NET-specific structure -- it is how the OS loader knows to hand off control to the CLR rather than executing the file as native code.
```

You can inspect the PE header of any assembly using:

```
dumpbin /headers MyAssembly.dll
```

For a detailed breakdown of each sub-structure and its fields, see [[The Operating System (Windows) File Header]].

---

## The CLR File Header

The **CLR file header** (sometimes called the **CLI header** or **COR20 header**) is the block of data that makes a PE file a *.NET assembly* rather than just a native executable. Without this header, the OS would attempt to execute the file as native machine code -- and fail, because the file contains CIL, not native instructions.

The CLR reads this header after the OS loader has mapped the PE file into memory. It contains the flags and pointers that the runtime needs to understand and manage the assembly:

| Field | Purpose |
|---|---|
| **Runtime version** | The minimum version of the CLR required to execute this assembly. |
| **Metadata directory** | A pointer (RVA and size) to the location of the type metadata and assembly manifest within the file. |
| **Resources directory** | A pointer to embedded managed resources. |
| **Entry point token** | For executable assemblies, the metadata token of the `Main` method (or whatever method is the entry point). For class libraries, this is zero. |
| **Strong name signature** | A pointer to the strong name hash, if the assembly is signed. |
| **Flags** | Bit flags indicating properties of the assembly. |

The **Flags** field is particularly important. Common flags include:

- `COMIMAGE_FLAGS_ILONLY` -- the assembly contains only CIL code, no mixed native/managed code
- `COMIMAGE_FLAGS_32BITREQUIRED` -- the assembly must run in a 32-bit process
- `COMIMAGE_FLAGS_STRONGNAMESIGNED` -- the assembly has been signed with a strong name

```ad-important
The combination of `ILONLY` and the absence of `32BITREQUIRED` is what allows an assembly to run as either 32-bit or 64-bit, depending on the host platform. This is the default for most .NET assemblies compiled with `AnyCPU`. If you see unexpected behavior when running on different architectures, checking these flags is a good first diagnostic step.
```

You can inspect the CLR header using:

```
dumpbin /clrheader MyAssembly.dll
```

For a detailed exploration of each field and its implications, see [[The CLR File Header]].

---

## CIL Code (Common Intermediate Language)

The **CIL code** (Common Intermediate Language, formerly called MSIL) is the actual compiled output of your C#, VB.NET, or F# source code -- but it is *not* native machine code. CIL is a **platform-agnostic, CPU-agnostic intermediate language** that sits between your high-level source code and the native instructions that your CPU actually executes.

This two-stage compilation model is fundamental to how .NET works:

```
Source Code (C#)  →  [C# Compiler]  →  CIL (in the assembly)
                                            ↓
                                     [JIT Compiler at runtime]
                                            ↓
                                     Native Machine Code (x86, x64, ARM, etc.)
```

At runtime, the **JIT (Just-In-Time) compiler** translates CIL into native instructions for the specific platform and CPU architecture the code is running on. This is what enables .NET's cross-platform capability -- the same CIL code in a single assembly can run on Windows x64, Linux ARM64, or macOS Apple Silicon, as long as a compatible .NET runtime is available.

Key characteristics of CIL:

- **Stack-based instruction set** -- CIL operates on an evaluation stack rather than registers. Values are pushed onto the stack, operations consume values from the stack, and results are pushed back.
- **Object-oriented** -- CIL has native instructions for creating objects, calling virtual methods, casting types, and throwing/catching exceptions.
- **Type-safe** -- the CLR can verify CIL code before JIT compilation to ensure it does not perform illegal memory access or type violations. This is called **verification**.
- **Stored in the `.text` section** of the PE file, alongside the metadata.

```ad-note
CIL is a fully specified language with its own syntax. You can write CIL directly using the `ilasm` (IL Assembler) tool, and you can disassemble any .NET assembly back to CIL using `ildasm`. In practice, you rarely write CIL by hand, but being able to read it is valuable for understanding compiler behavior, diagnosing performance issues, or investigating exactly what your code compiles to.
```

For a deeper look at CIL instructions and how they relate to metadata, see [[CIL Code, Type Metadata, and the Assembly Manifest]].

---

## Type Metadata

Every .NET assembly contains a complete, machine-readable description of **every type** defined within it. This description is called **type metadata**, and it is one of the most powerful features of the .NET platform.

Type metadata includes:

- **Type definitions** -- the name, visibility, base class, and implemented interfaces of every class, struct, enum, interface, and delegate in the assembly
- **Member definitions** -- every method, property, field, event, and constructor, along with their signatures (parameter types, return types, access modifiers)
- **Custom attributes** -- all attributes applied to types and members (e.g., `[Serializable]`, `[Obsolete]`, `[DllImport]`)
- **Generic type parameters** -- constraints and variance annotations for generic types and methods
- **Nested type relationships** -- which types are nested inside other types

The CLR and the broader .NET ecosystem rely on type metadata for many purposes:

| Consumer | How It Uses Type Metadata |
|---|---|
| **CLR (runtime)** | Resolves method calls, lays out objects in memory, enforces access modifiers and type safety |
| **JIT compiler** | Determines the size and layout of types to generate correct native code |
| **Reflection** | Powers `typeof()`, `GetType()`, `GetMethods()`, and the entire `System.Reflection` namespace |
| **Serialization** | Frameworks like `System.Text.Json` and `System.Xml.Serialization` read metadata to know how to serialize/deserialize objects |
| **IntelliSense / IDE** | Visual Studio and other editors read metadata to provide autocomplete, parameter info, and documentation tooltips |
| **Compilers** | When you add a reference to an assembly, the compiler reads its type metadata to validate your code against the available types and members |

```ad-warning
Type metadata and the **assembly manifest** are both metadata, but they describe different things. Type metadata describes the **types** inside the assembly (classes, methods, properties). The assembly manifest describes the **assembly itself** (name, version, dependencies). It is a common point of confusion to conflate the two.
```

The type metadata is stored in a set of **metadata tables** within the assembly -- structured binary tables with well-defined schemas for type definitions, method definitions, field definitions, and so on. Tools like `ildasm` and ILSpy parse these tables to show you the contents of an assembly.

For a detailed look at how type metadata is structured and how it relates to CIL code, see [[CIL Code, Type Metadata, and the Assembly Manifest]].

---

## Assembly Manifest

The **assembly manifest** is a block of metadata that describes *the assembly itself* -- its identity, its version, and its relationship to other assemblies. If type metadata answers the question *"What types does this assembly contain?"*, the manifest answers the question *"What **is** this assembly, and what does it need?"*

Every .NET assembly contains exactly one manifest. The manifest includes:

| Manifest Entry | Purpose |
|---|---|
| **Assembly name** | The simple name of the assembly (e.g., `System.Collections`). |
| **Version number** | The four-part version (`major.minor.build.revision`) that is part of the assembly's identity. |
| **Culture / locale** | For [[Satellite Assemblies]], this specifies the language/region (e.g., `en-US`, `fr-FR`). For culture-neutral assemblies, this is empty. |
| **Public key / public key token** | If the assembly is strong-name signed, the public key (or its 8-byte hash token) is recorded here. This is part of the assembly's identity. |
| **List of referenced assemblies** | Every external assembly that this assembly depends on, including the expected version and public key token of each dependency. |
| **List of files** | For multi-module assemblies, a list of all modules that make up the assembly. In modern .NET, virtually all assemblies are single-module, so this typically lists only the assembly itself. |
| **Exported types** | For multi-module assemblies, types that are defined in secondary modules but exposed by the primary module. Rarely relevant in modern .NET. |

The CLR uses the manifest at load time to:

1. **Verify the assembly's identity** -- confirm the name, version, culture, and public key match what the referencing assembly expected
2. **Resolve dependencies** -- locate and load the external assemblies listed in the manifest
3. **Enforce version policy** -- apply binding redirects or publisher policy if configured

```ad-note
The manifest is what makes assemblies **self-describing**. Unlike the legacy COM model, where type information and registration data were stored in the Windows registry, a .NET assembly carries everything the runtime needs to identify it and resolve its dependencies. This is why .NET does not require registry entries and why `xcopy` deployment (just copying files) works.
```

For a detailed exploration of manifest contents and how to read them, see [[CIL Code, Type Metadata, and the Assembly Manifest]].

---

## Optional Embedded Resources

Assemblies can contain **embedded resources** -- arbitrary binary data that is compiled directly into the assembly file. These resources are baked in at build time and become part of the assembly's physical file on disk.

Common examples of embedded resources:

- **Images and icons** -- application icons, toolbar images, splash screens
- **String tables** -- for localization via `.resx` resource files
- **XML or JSON files** -- embedded configuration, schemas, templates
- **Other files** -- certificates, license text, data files

Embedded resources are accessed at runtime through the `System.Reflection.Assembly` class:

```csharp
// Get a stream to an embedded resource
Assembly assembly = Assembly.GetExecutingAssembly();
Stream stream = assembly.GetManifestResourceStream("MyApp.Resources.logo.png");

// List all embedded resources in an assembly
string[] resourceNames = assembly.GetManifestResourceNames();
```

```ad-note
**Satellite assemblies** are a special case of assemblies that contain *only* embedded resources (no CIL code). They are used for **localization** -- each satellite assembly contains the translated strings and culture-specific resources for a single locale, and the runtime automatically loads the correct one based on the current thread's culture. See [[Satellite Assemblies]] for details.
```

Embedded resources live in the `.rsrc` section of the PE file (for Win32-level resources like the application icon) or in the managed resources area pointed to by the CLR header (for managed resources accessed via `GetManifestResourceStream()`).

---

## How It All Fits Together

The following diagram shows the full lifecycle of how the different layers of a .NET assembly are consumed, from the moment you double-click an `.exe` to the moment your code runs:

```
 YOU double-click MyApp.exe
        │
        ▼
 ┌─────────────────────────────────────┐
 │  1. OS Loader reads PE/COFF Header  │
 │     - Validates the file            │
 │     - Maps sections into memory     │
 │     - Finds CLR data directory      │
 │     - Hands off to CLR bootstrap    │
 └──────────────┬──────────────────────┘
                │
                ▼
 ┌─────────────────────────────────────┐
 │  2. CLR reads CLR File Header       │
 │     - Checks runtime version        │
 │     - Locates metadata              │
 │     - Locates entry point token     │
 │     - Reads flags (IL-only, etc.)   │
 └──────────────┬──────────────────────┘
                │
                ▼
 ┌─────────────────────────────────────┐
 │  3. CLR reads Assembly Manifest     │
 │     - Identifies the assembly       │
 │     - Resolves external references  │
 │     - Loads dependent assemblies    │
 └──────────────┬──────────────────────┘
                │
                ▼
 ┌─────────────────────────────────────┐
 │  4. CLR reads Type Metadata         │
 │     - Resolves the entry point      │
 │       method from its token         │
 │     - Lays out types as needed      │
 └──────────────┬──────────────────────┘
                │
                ▼
 ┌─────────────────────────────────────┐
 │  5. JIT compiles CIL → native code  │
 │     - Compiles methods on first     │
 │       call (or ahead-of-time with   │
 │       ReadyToRun / NativeAOT)       │
 └──────────────┬──────────────────────┘
                │
                ▼
 ┌─────────────────────────────────────┐
 │  6. Native code executes            │
 │     - Your program is running       │
 └─────────────────────────────────────┘
```

Each layer depends on the one above it. The OS must load the file before the CLR can read it. The CLR must read its header before it can find the metadata. The metadata must be parsed before the JIT can compile CIL. This layered design is what allows .NET to sit on top of different operating systems -- only the outermost PE layer is platform-specific.

---

## Inspecting an Assembly

You do not need to take any of this on faith -- .NET provides several tools that let you inspect every layer of an assembly directly:

| Tool | What It Shows | Platform |
|---|---|---|
| `dumpbin /headers` | The full PE/COFF header -- DOS header, COFF header, optional header, section table | Windows (ships with Visual Studio) |
| `dumpbin /clrheader` | The CLR file header -- runtime version, flags, entry point token, metadata pointers | Windows (ships with Visual Studio) |
| `ildasm` | CIL code, type metadata, and the assembly manifest in a tree view GUI. The classic .NET disassembler. | Windows (.NET Framework SDK) |
| `dotnet-ildasm` | A cross-platform alternative to `ildasm`, available as a .NET global tool | Cross-platform |
| **ILSpy** | Full-featured open-source decompiler. Shows CIL, decompiled C#, metadata, manifest, and resources. | Windows (GUI), cross-platform (CLI) |
| **dotPeek** | JetBrains' free decompiler with features similar to ILSpy | Windows |

```ad-note
`ildasm` is particularly useful for learning because it shows the raw CIL instructions alongside the metadata tables. If you want to understand exactly what the compiler generated from your C# code, `ildasm` (or ILSpy's "IL" view) is the tool to use.
```

---

## Summary

| Element | What It Describes | Who Reads It | Key Purpose |
|---|---|---|---|
| **PE/COFF File Header** | The physical binary file structure | OS loader | Validate and map the file into memory |
| **CLR File Header** | .NET-specific flags and pointers | CLR | Identify the file as managed, locate metadata and entry point |
| **CIL Code** | The compiled intermediate language | JIT compiler | Platform-agnostic code that gets compiled to native instructions |
| **Type Metadata** | Every type and member in the assembly | CLR, reflection, tooling, compilers | Runtime type resolution, memory layout, IntelliSense, serialization |
| **Assembly Manifest** | The assembly's identity and dependencies | CLR | Resolve references, enforce versioning, enable self-description |
| **Embedded Resources** | Binary assets (images, strings, files) | Application code via reflection | Bundle assets directly into the assembly |

Together, these six elements make a .NET assembly a **self-contained, self-describing, platform-portable unit** of deployment. The OS header makes it loadable. The CLR header makes it managed. The CIL makes it portable. The metadata makes it inspectable. The manifest makes it identifiable. And the resources make it complete.

For the roles that assemblies play in the broader .NET ecosystem, see [[The Role of .NET Assemblies]].
