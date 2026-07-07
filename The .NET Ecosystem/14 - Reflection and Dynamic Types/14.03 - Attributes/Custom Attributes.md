---
tags:
 - csharp
 - attributes
---

## Creating Custom Attributes

A custom attribute is a class that extends `System.Attribute`. You control where it can be applied using [[AttributeUsage]].

### Step 1: Define the Attribute Class

```csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, AllowMultiple = false)]
public class AuthorAttribute : Attribute
{
    public string Name { get; }            // set via constructor (positional, required)
    public string Version { get; set; }    // set via named property (optional)

    public AuthorAttribute(string name)    // constructor params = positional args
    {
        Name = name;
    }
}
```

Rules for attribute classes:
- Must inherit from `System.Attribute` (directly or indirectly)
- Convention: suffix the class name with `Attribute` (compiler lets you omit it when applying)
- Constructor parameters become **positional** (required) arguments
- Public writable properties become **named** (optional) arguments -- see [[Named Property Syntax]]

### Step 2: Apply It

```csharp
[Author("Long", Version = "1.0")]   // "Long" = positional, Version = named
public class MyService
{
    [Author("Pham")]                 // Version omitted -- it's optional
    public void Process() { }
}
```

### Step 3: Read It via Reflection

```csharp
Type t = typeof(MyService);

// Generic approach (.NET 4.5+)
var attr = t.GetCustomAttribute<AuthorAttribute>();
Console.WriteLine($"{attr.Name}, v{attr.Version}");  // "Long, v1.0"

// Check if applied without retrieving
bool hasAuthor = t.IsDefined(typeof(AuthorAttribute));

// Multiple attributes (when AllowMultiple = true)
var all = t.GetCustomAttributes<AuthorAttribute>();
```

See [[Reflecting on Attributes Using Late Binding and Early Binding]] for the full reflection patterns.

## Positional vs Named Arguments

| | Positional | Named |
|---|---|---|
| Defined by | Constructor parameter | Public writable property or field |
| Required? | Yes | No |
| Syntax | `[Author("Long")]` | `[Author("Long", Version = "1.0")]` |

## What Can Be an Attribute Parameter?

Attribute constructor and property types are restricted to:
- Primitives (`int`, `bool`, `string`, `float`, `double`, etc.)
- `Type`
- Enums
- One-dimensional arrays of the above

```ad-warning
You **cannot** use complex objects, `List<T>`, dictionaries, or other reference types as attribute parameters. This is a CLR metadata limitation -- attribute data must be expressible as compile-time constants baked into IL metadata.
```

## See Also

- [[Attribute Overview]]
- [[AttributeUsage]]
- [[AllowMultiple]]
- [[Named Property Syntax]]
- [[Reflecting on Attributes Using Late Binding and Early Binding]]
