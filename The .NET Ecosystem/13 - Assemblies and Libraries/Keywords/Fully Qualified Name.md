---
tags:
 - csharp
 - namespaces
---

## What Is a Fully Qualified Name?

A **fully qualified type name** uniquely identifies a type by combining several components:

| Component | Example |
|---|---|
| Assembly | `MyLibrary.dll` |
| Namespace | `MyCompany.Data` |
| Type Name | `CustomerRepository` |

Full form: `MyCompany.Data.CustomerRepository, MyLibrary`

In code, the namespace-qualified name is usually sufficient because the assembly reference is already established via project references:

```csharp
// Namespace-qualified (most common in C# code):
System.Collections.Generic.List<int> list = new();

// Usually shortened with a using directive:
using System.Collections.Generic;
List<int> list = new();
```

## In CIL (Common Intermediate Language)

In the compiled IL code, types are always referenced with their fully qualified name including the assembly:

```
[System.Runtime]System.Collections.Generic.List`1<int32>
```

This is how the runtime resolves types — the assembly name disambiguates types that share the same namespace and name across different libraries.

```ad-info
In CIL code, types are always defined with the fully qualified name including the assembly reference. This is what makes it possible for two assemblies to contain identically named types without conflict.
```

## See Also
- [[Aliases]]
- [[The Role of .NET Assemblies]]
- [[Nested Namespace]]
