---
tags:
 - csharp
 - attributes
---

## AllowMultiple

The `AllowMultiple` property on [[AttributeUsage]] controls whether an attribute can be applied more than once to the same code element. Default is `false`.

---

## AllowMultiple = false (Default)

```csharp
[AttributeUsage(AttributeTargets.Class, AllowMultiple = false)]
public class AuthorAttribute : Attribute
{
    public string Name { get; }
    public AuthorAttribute(string name) => Name = name;
}

[Author("Long")]
[Author("Pham")]  // CS0579: Duplicate 'Author' attribute — compiler error
public class MyClass { }
```

---

## AllowMultiple = true

```csharp
[AttributeUsage(AttributeTargets.Class, AllowMultiple = true)]
public class TagAttribute : Attribute
{
    public string Value { get; }
    public TagAttribute(string value) => Value = value;
}

[Tag("important")]
[Tag("reviewed")]
[Tag("v2")]        // all three are applied
public class MyClass { }
```

Reading multiple instances via reflection:

```csharp
Type t = typeof(MyClass);
var tags = t.GetCustomAttributes<TagAttribute>();
foreach (var tag in tags)
    Console.WriteLine(tag.Value);
// important
// reviewed
// v2
```

---

## When to Use AllowMultiple

| AllowMultiple    | Use when                                    | Example                          |
| ---------------- | ------------------------------------------- | -------------------------------- |
| `false` (default) | Attribute represents a single piece of metadata | `[Author]`, `[Table]`, `[Route]` |
| `true`           | Multiple instances make sense               | `[Tag]`, `[Authorize]`, `[Header]` |

---

## See Also

- [[AttributeUsage]]
- [[Custom Attributes]]
