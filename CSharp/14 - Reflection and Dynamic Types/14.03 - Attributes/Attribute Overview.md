---
tags:
 - csharp
 - attributes
---

## What Are Attributes?

**Attributes** are code annotations that embed additional metadata into an assembly. They are class types that extend `System.Attribute`. You apply them to types, members, assemblies, or modules using square bracket syntax.

```csharp
[Serializable]           // attribute on a class
public class Person
{
    [Obsolete("Use FullName instead")]   // attribute on a property
    public string Name { get; set; }

    [return: MaybeNull]  // attribute on a return value
    public string? FindById(
        [NotNull] int id)  // attribute on a parameter
    { }
}
```

## How Attributes Work

Attributes don't *do* anything by themselves. When you apply an attribute, it simply writes metadata into the assembly. That metadata is completely inert until some other code reflects over it.

Three participants in the attribute pattern:
1. **The attribute class** -- defines what metadata can be stored
2. **The annotated element** -- the code wearing the attribute
3. **The consumer** -- code that reads the attribute via [[Reflecting on Attributes Using Late Binding and Early Binding|reflection]] and acts on it

```
[MyAttribute("data")]        ->  Compiler writes to metadata  ->  Reflection reads it  ->  Consumer acts on it
 on class/method/etc.             (baked into .dll)              GetCustomAttributes()    (framework, tool, your code)
```

Common consumers: serializers, DI containers, ORMs, test frameworks, compilers (for [[Built-in Attributes|built-in attributes]]).

```ad-note
Attributes are essentially useless until something explicitly reflects over them. If nothing reads the metadata, the attribute is ignored and completely harmless.
```

### Attributes Are Not Allocated Until Reflected Upon

When you apply an attribute, **no constructor runs and no object is created**. The compiler stores the constructor arguments as raw bytes in the metadata tables — nothing more.

```csharp
[Author("Long")]          // compiler does NOT call new AuthorAttribute("Long")
public class MyClass { }  // just writes "Long" as bytes in metadata
```

The `AuthorAttribute` constructor only executes when something reflects:

```csharp
// NOW the CLR reads the metadata, calls new AuthorAttribute("Long"), returns the object
var attr = typeof(MyClass).GetCustomAttribute<AuthorAttribute>();
```

```
COMPILE TIME                  IN THE .DLL                      RUNTIME

[Author("Long")]              Metadata:                        GetCustomAttribute<Author>()
                              "Long" as raw bytes                  │
No constructor.               No object.                       CLR reads metadata
No allocation.                Just data in a table.            Calls constructor
                                                               Returns live object
```

This is why attribute parameters are restricted to compile-time constants (primitives, strings, `Type`, enums) — they must be storable as raw bytes in metadata, not as live objects.


## Attribute Syntax

```csharp
// Shorthand (omit the "Attribute" suffix -- compiler resolves it)
[Serializable]
[Obsolete("Use NewMethod")]

// Full name (also valid)
[SerializableAttribute]
[ObsoleteAttribute("Use NewMethod")]

// Multiple attributes on one line
[Serializable, Obsolete("Old")]

// Explicit targets -- tells the compiler exactly what the attribute applies to
[assembly: InternalsVisibleTo("Tests")]
[return: NotNull]
[field: NonSerialized]
```

The shorthand works because the compiler automatically appends `Attribute` to the name when resolving the type. `[Serializable]` and `[SerializableAttribute]` compile to identical IL.

## See Also

- [[Custom Attributes]]
- [[Built-in Attributes]]
- [[AttributeUsage]]
- [[Reflecting on Attributes Using Late Binding and Early Binding]]
