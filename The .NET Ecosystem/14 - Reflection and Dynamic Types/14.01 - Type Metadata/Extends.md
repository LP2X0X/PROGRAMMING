---
tags:
 - csharp
 - reflection
 - metadata
---

## The Extends Metadata Token

The **Extends** token documents the base type (parent class) of a given type. Every class in .NET inherits from something — if you don't specify a base class, the compiler automatically sets Extends to `System.Object`.

This is how the runtime knows the inheritance chain at load time, *before* any of your code runs.

### Example: Explicit Base Class

```csharp
// C# code
public class SportsCar : Car { }
```

```il
// IL metadata — explicitly extends Car
.class public auto ansi beforefieldinit SportsCar
    extends CarLibrary.Car
```

### Example: Implicit Base Class

```csharp
// C# code — no base class specified
public class Person { }
```

```il
// IL metadata — compiler inserts System.Object
.class public auto ansi beforefieldinit Person
    extends [System.Runtime]System.Object
```

### Special Cases

Different C# type categories produce different Extends values in the metadata:

| C# Type | Extends in Metadata | Why |
|---|---|---|
| Regular class (no base) | `System.Object` | Compiler inserts the implicit base |
| Regular class with base | The specified base class | Direct inheritance |
| `struct` | `System.ValueType` | All value types derive from ValueType |
| `enum` | `System.Enum` | Enums are specialized value types |
| `interface` | *(no Extends)* | Interfaces don't have a base type in metadata |

```ad-info
When the Extends token references a type from an external assembly, you'll see the assembly prefix in brackets — e.g., `[System.Runtime]System.Object`. This tells you the base type is resolved via an [[AssemblyRef]] to that external assembly.
```

### Why This Matters for Reflection

When you call `typeof(SportsCar).BaseType` in C#, the reflection API reads the Extends token from the metadata to return the base type. The entire inheritance chain — from your class up to `System.Object` — is built by following the Extends tokens at each level.

## See Also

- [[TypeDef and TypeRef]]
- [[Field]]
- [[Assembly]]
