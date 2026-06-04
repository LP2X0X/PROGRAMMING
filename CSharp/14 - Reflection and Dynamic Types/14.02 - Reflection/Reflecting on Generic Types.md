---
tags:
 - csharp
 - reflection
 - generics
---

## Reflecting on Generic Types

Generic types require special syntax and handling when working with reflection because the type name in metadata uses a **backtick notation**, not the angle-bracket syntax you write in C#.

## The Backtick Naming Convention

In metadata, generic types are named with a backtick (`` ` ``) followed by the number of type parameters:

| C# Syntax | Metadata Name | Backtick Count |
| --- | --- | --- |
| `List<T>` | `System.Collections.Generic.List`1` | 1 (one type param) |
| `Dictionary<TKey, TValue>` | `System.Collections.Generic.Dictionary`2` | 2 (two type params) |
| `Tuple<T1, T2, T3>` | `System.Tuple`3` | 3 |
| `Action<T1, T2>` | `System.Action`2` | 2 |

```csharp
// Getting an open generic type via string name
Type listOpen = Type.GetType("System.Collections.Generic.List`1");
Console.WriteLine(listOpen); // System.Collections.Generic.List`1

Type dictOpen = Type.GetType("System.Collections.Generic.Dictionary`2");
Console.WriteLine(dictOpen); // System.Collections.Generic.Dictionary`2
```

## Open vs Closed Generic Types

| Term | Meaning | Example |
| --- | --- | --- |
| **Open generic** | Type parameters not yet specified | `List<>` / `List`1` |
| **Closed generic** | Type parameters filled in | `List<int>` / `List`1[System.Int32]` |

```csharp
// Open generic — via typeof with empty angle brackets
Type openList = typeof(List<>);
Console.WriteLine(openList.IsGenericTypeDefinition); // True
Console.WriteLine(openList.ContainsGenericParameters); // True

// Closed generic — via typeof with concrete type
Type closedList = typeof(List<int>);
Console.WriteLine(closedList.IsGenericTypeDefinition); // False
Console.WriteLine(closedList.ContainsGenericParameters); // False
```

## Constructing a Closed Type from an Open One

This is one of the most powerful patterns in generic reflection — building a concrete type at runtime from an open definition:

```csharp
Type openDict = typeof(Dictionary<,>);

// Make it concrete at runtime
Type closedDict = openDict.MakeGenericType(typeof(string), typeof(int));

// Now you can create instances
object instance = Activator.CreateInstance(closedDict);
// instance is a Dictionary<string, int>
```

```ad-warning
title: Argument Count Must Match
`MakeGenericType` throws `ArgumentException` if the number of types you pass doesn't match the number of type parameters on the open generic. Always check `GetGenericArguments().Length` if you're working with unknown types.
```

## Inspecting Type Parameters

```csharp
Type t = typeof(Dictionary<string, int>);

// Get the generic type arguments
Type[] args = t.GetGenericArguments();
foreach (var arg in args)
    Console.WriteLine(arg.Name); // String, Int32

// Get back to the open generic definition
Type openDef = t.GetGenericTypeDefinition();
Console.WriteLine(openDef); // Dictionary`2
```

## Useful Properties for Generic Reflection

| Property / Method | What It Tells You |
| --- | --- |
| `IsGenericType` | Is this a generic type at all? |
| `IsGenericTypeDefinition` | Is this an open generic (`List<>`)? |
| `ContainsGenericParameters` | Are there unresolved type parameters? |
| `GetGenericArguments()` | Returns the type parameters / arguments |
| `GetGenericTypeDefinition()` | Returns the open generic from a closed one |
| `MakeGenericType(params Type[])` | Creates a closed generic from an open one |

## See Also

- [[Reflection Overview]]
- [[Reflecting on Static Types]]
- [[Reflection Usage]]
