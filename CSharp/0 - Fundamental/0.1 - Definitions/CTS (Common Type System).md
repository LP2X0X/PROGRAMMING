---
tags: 
 - csharp
 - definition
---

It's called "Common" because it's the **one shared type system that all .NET languages agree on**.

### The problem it solves

Before .NET, different languages had different ideas about types:

```
C++:        int = 32-bit, long = 32-bit (on Windows)
VB6:        Integer = 16-bit, Long = 32-bit
Java:       int = 32-bit, long = 64-bit

What happens when C++ code passes a "long" to VB code?
They don't agree on what "long" means. Chaos.
```

### The solution: one common system

The CTS defines **one set of rules** that all .NET languages must follow:

```
CTS defines:        System.Int32  (always 32-bit, everywhere, every language)

C# calls it:        int           → maps to System.Int32
VB.NET calls it:    Integer       → maps to System.Int32
F# calls it:        int           → maps to System.Int32

Different names, same underlying type. They all compile to the same IL.
```

```
C# code:            int x = 42;
VB.NET code:        Dim x As Integer = 42
F# code:            let x : int = 42

All three compile to exactly:
  IL:               .locals init (int32 x)
                    ldc.i4 42
                    stloc.0
```

### What the CTS defines

```
1. VALUE TYPES (stored on stack, copied by value)
   ├── int, float, double, bool, decimal, struct, enum
   └── System.Int32, System.Boolean, System.DateTime...

2. REFERENCE TYPES (stored on heap, copied by reference)
   ├── class, interface, delegate, array, string
   └── System.Object, System.String, System.Array...

3. RULES
   ├── All types inherit from System.Object
   ├── Single inheritance for classes
   ├── Multiple inheritance for interfaces
   ├── How generics work
   ├── How access modifiers work (public, private...)
   └── How type conversions work
```

### Why "Common" matters — cross-language interop

```csharp
// C# library
public class InspectionResult
{
    public int DefectCount { get; set; }        // System.Int32
    public string BoardId { get; set; }          // System.String
    public List<Defect> Defects { get; set; }    // System.Collections.Generic.List<T>
}
```

```vb
' VB.NET code can use the C# class directly — types are identical
Dim result As New InspectionResult()
result.DefectCount = 5          ' Integer → System.Int32, same thing
result.BoardId = "PCB-001"      ' String → System.String, same thing
```

```fsharp
// F# can also use it — same types underneath
let result = InspectionResult()
result.DefectCount <- 5         // int → System.Int32
```

This works because all three languages compile to IL using CTS types. They don't need to "translate" — they already speak the same type language.

### The full picture: CTS sits inside the CLR

```
Your Code (C#, VB, F#)
    │
    ▼
Compiler (Roslyn, etc.)
    │
    ▼
IL Code ──── uses CTS types (System.Int32, System.String, etc.)
    │
    ▼
CLR (Common Language Runtime)
  ├── CTS  — the shared type system (what types exist, how they behave)
  ├── CLS  — Common Language Specification (subset all languages must support)
  ├── JIT  — compiles IL to machine code (what you learned earlier)
  └── GC   — garbage collector (manages memory)
    │
    ▼
Machine Code → CPU
```

### CTS vs CLS — quick distinction

```
CTS = everything .NET supports (full type system)
CLS = the minimum subset all languages must support

Example:
  C# supports unsigned integers (uint)     ← CTS allows this
  VB.NET also supports uint                 ← fine
  Some .NET languages may NOT support uint  ← not in CLS

  If you want your library usable by ALL .NET languages:
    → stick to CLS-compliant types only
    → mark it with [assembly: CLSCompliant(true)]
```

### One-line summary

> It's called "Common" because it's the **shared contract** that makes every .NET language's types interchangeable — one type system to rule them all.
