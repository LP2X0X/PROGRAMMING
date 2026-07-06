---
tags:
 - csharp
 - namespaces
---

## Using Aliases

The `using` directive can create an **alias** for a type's fully qualified name. The alias is substituted at compile time — it is purely syntactic sugar and produces no runtime overhead.

### Resolving Name Clashes

When two namespaces contain a type with the same name, you get an ambiguity error. Aliases let you disambiguate:

```csharp
using System;
using MyCompany.Utilities;  // also has a Console class

// Ambiguous — which Console?
// Console.WriteLine("Hi");  // CS0104 compiler error

// Fix with alias:
using SysConsole = System.Console;
using MyConsole = MyCompany.Utilities.Console;

SysConsole.WriteLine("Using System.Console");
MyConsole.Log("Using MyCompany.Utilities.Console");
```

### Aliasing Long Type Names

```csharp
using Rppr = MyCompany.Logistics.Relocation.RelocatedPlantProductReleaseItem;

Rppr item = new Rppr();
```

### Global Using Aliases (C# 10+)

```csharp
// In a GlobalUsings.cs file — applies to the entire project
global using Env = System.Environment;
```

### Using Aliases for Any Type (C# 12+)

C# 12 expanded aliases beyond named types — you can now alias tuples, arrays, pointers, and generic types:

```csharp
using Point = (int X, int Y);
using NumberList = System.Collections.Generic.List<int>;

Point p = (10, 20);
NumberList nums = [1, 2, 3];
```

```ad-warning
title: Alias Scope
A standard `using` alias is file-scoped — it only applies within the file where it is declared. Use `global using` if you need the alias available across the entire project.
```

## See Also
- [[Fully Qualified Name]]
- [[Nested Namespace]]
