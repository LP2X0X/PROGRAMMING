---
tags:
 - csharp
 - reflection
 - metadata
---

## TypeDef and TypeRef Metadata Tokens

These are the two most fundamental metadata tokens — they describe every type your assembly knows about, whether it was defined locally or comes from an external dependency.

### TypeDef — Types Defined Here

Every type defined within the current assembly gets a **TypeDef** token (`TypeDef #n`). This is the *full* definition — all fields, methods, properties, events, implemented interfaces, and base type are nested under it.

```il
// A class defined in your assembly:
.class public auto ansi beforefieldinit MyApp.Person
    extends [System.Runtime]System.Object
{
    .field private string _name
    .method public hidebysig instance string get_Name() cil managed { ... }
    .method public hidebysig specialname rtspecialname
            instance void .ctor() cil managed { ... }
}
```

The TypeDef contains everything the runtime needs to lay out the type in memory, resolve method calls, and enforce access rules.

```ad-info
Every assembly has at least one TypeDef: the compiler-generated `<Module>` type, which holds module-level (global) methods and fields. You'll see it as `TypeDef #1` in `ildasm.exe` output even if you didn't define any global members.
```

### TypeRef — Types Referenced from External Assemblies

When your code uses a type from another assembly (e.g., `System.Console`, `System.String`), the metadata stores a **TypeRef** token — a pointer to that type's full definition in the external assembly.

```il
// TypeRef pointing to System.Console in an external assembly:
[System.Console]System.Console
```

The TypeRef doesn't contain the full type info. It records:

- The **type name** and **namespace**
- Which [[AssemblyRef]] to resolve it from

At runtime, the CLR follows the TypeRef to the external assembly, finds the matching TypeDef there, and loads the full definition.

### TypeDef vs TypeRef

| | TypeDef | TypeRef |
|---|---|---|
| **What it describes** | A type *you* wrote (in this assembly) | A type from *another* assembly |
| **Contains full definition?** | Yes — fields, methods, base type, everything | No — just a reference pointer |
| **Example** | Your `Person` class | `System.String`, `System.Console` |
| **Analogy** | The actual class file | An `import` / `using` statement |

### How They Work Together

When the compiler sees your code using `System.Console.WriteLine(...)`, it:

1. Emits an [[AssemblyRef]] for `System.Console` (the assembly)
2. Emits a **TypeRef** for `System.Console` (the type) pointing to that AssemblyRef
3. Emits a **MemberRef** for `WriteLine` pointing to that TypeRef

At runtime, the CLR follows this chain: MemberRef -> TypeRef -> AssemblyRef -> loads the assembly -> finds the TypeDef -> resolves the method.

```ad-note
You can count the TypeDef tokens to see how many types your assembly defines, and the TypeRef tokens to see how many external types it depends on. This is a useful way to gauge the surface area of your code and its coupling to external libraries.
```

## See Also

- [[Assembly]]
- [[AssemblyRef]]
- [[Extends]]
- [[Field]]
- [[User Strings]]
- [[How to generate the IL file of an assembly]]
