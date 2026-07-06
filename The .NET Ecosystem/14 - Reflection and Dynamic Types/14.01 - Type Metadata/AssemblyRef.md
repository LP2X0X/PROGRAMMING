---
tags:
 - csharp
 - reflection
 - metadata
---

## The AssemblyRef Metadata Token

For every external assembly your code depends on, the compiler emits an **AssemblyRef token**. This records the identity of the referenced assembly so the runtime can locate and load the correct version at load time.

Every .NET assembly references at least `System.Runtime` (the base class library), so you will always see at least one AssemblyRef — even in the simplest program.

### Example ildasm Output

```il
.assembly extern System.Runtime
{
  .publickeytoken = (B0 3F 5F 7F 11 D5 0A 3A)
  .ver 8:0:0:0
}

.assembly extern System.Console
{
  .publickeytoken = (B0 3F 5F 7F 11 D5 0A 3A)
  .ver 8:0:0:0
}
```

Each AssemblyRef contains:

- **Assembly name** — the simple name of the dependency
- **Version number** — the expected `major.minor.build.revision`
- **Public key token** — for strong-named assemblies, used to verify identity

### How the Runtime Uses AssemblyRef

The runtime uses AssemblyRef entries during **assembly resolution** — the process of finding the right DLL at load time. It:

1. Reads the AssemblyRef name and version
2. Searches the probing paths (app directory, GAC, NuGet package cache, etc.)
3. Verifies the public key token matches (for strong-named assemblies)
4. Loads the assembly and makes its types available via the corresponding [[TypeDef and TypeRef|TypeRef]] tokens

```ad-note
The number of AssemblyRef tokens tells you exactly how many external assemblies your code depends on. This is a quick way to audit dependencies — open `ildasm.exe` and count them.
```

### Assembly vs AssemblyRef

| Token | Describes | Purpose |
|---|---|---|
| [[Assembly]] | The current assembly | "This is who I am" |
| AssemblyRef | An external dependency | "This is who I need" |

Think of it like imports in a module system: the [[Assembly]] token is the module declaration, and each AssemblyRef is an import statement pointing to another module.

## See Also

- [[Assembly]]
- [[TypeDef and TypeRef]]
