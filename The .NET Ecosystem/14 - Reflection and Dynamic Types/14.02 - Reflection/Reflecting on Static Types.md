---
tags:
 - csharp
 - reflection
---

## Reflecting on Static Types

Static types (classes marked with the `static` keyword) require special handling when using reflection because they cannot be instantiated.

### Getting the Type

You cannot use `Type.GetType()` with just a simple type name for static types and expect it to resolve without an assembly-qualified name. Instead, the simplest approach is the `typeof` operator:

```csharp
// Works — typeof is a compile-time operator
Type t = typeof(Console);
Type t2 = typeof(Math);
Type t3 = typeof(Environment);

// Also works — fully qualified name with assembly
Type t4 = Type.GetType("System.Console, System.Console");
```

```ad-note
title: Common Misconception
`Type.GetType(string)` works for static types — the issue is usually a missing assembly-qualified name, not the `static` modifier. `typeof` is simpler and compile-time safe.
```

### What You Can Do

Since you can't instantiate a static class, you skip `Activator.CreateInstance` and go straight to invoking static members:

```csharp
Type mathType = typeof(Math);

// Get a static method
MethodInfo sqrt = mathType.GetMethod("Sqrt");
object result = sqrt.Invoke(null, new object[] { 16.0 });
// null for the instance — static methods have no "this"
Console.WriteLine(result); // 4

// Get a static property
Type envType = typeof(Environment);
PropertyInfo machineName = envType.GetProperty("MachineName");
Console.WriteLine(machineName.GetValue(null)); // null — static property
```

### Detecting Static Types via Reflection

A static class in C# is compiled as `abstract sealed` in IL. You can check:

```csharp
Type t = typeof(Math);
bool isStatic = t.IsAbstract && t.IsSealed;
Console.WriteLine($"Math is static: {isStatic}"); // True
```

| C# Keyword | IL Flags | `IsAbstract` | `IsSealed` |
| --- | --- | --- | --- |
| `static class` | `abstract sealed` | `true` | `true` |
| `abstract class` | `abstract` | `true` | `false` |
| `sealed class` | `sealed` | `false` | `true` |
| regular class | (none) | `false` | `false` |

## See Also

- [[Reflection Overview]]
- [[Reflection Usage]]
- [[Reflecting on Generic Types]]
