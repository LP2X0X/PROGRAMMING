---
tags:
 - csharp
 - reflection
 - attributes
---

## Reading Attributes via Reflection

Attributes are metadata stored in the assembly. Reflection is how you read them at runtime. There are two approaches: **early binding** (attribute type known at compile time) and **late binding** (attribute type discovered at runtime).

## Defining and Applying a Custom Attribute

```csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method)]
public class DescriptionAttribute : Attribute
{
    public string Text { get; }
    public DescriptionAttribute(string text) => Text = text;
}

[Description("A sample class")]
public class MyClass
{
    [Description("A sample method")]
    public void MyMethod() { }
}
```

## Early Binding -- Type Known at Compile Time

You have a compile-time reference to the attribute type, so you can use generics and cast directly:

```csharp
Type t = typeof(MyClass);

// Read class-level attribute -- generic overload returns typed result
var classAttr = t.GetCustomAttribute<DescriptionAttribute>();
Console.WriteLine(classAttr.Text); // "A sample class"

// Read method-level attribute
MethodInfo m = t.GetMethod("MyMethod");
var methodAttr = m.GetCustomAttribute<DescriptionAttribute>();
Console.WriteLine(methodAttr.Text); // "A sample method"

// Check if an attribute exists without retrieving it
bool hasDesc = t.IsDefined(typeof(DescriptionAttribute));
```

This is the approach you'll use 90% of the time -- whenever you own or reference the attribute assembly.

## Late Binding -- Type Discovered at Runtime

When you load assemblies dynamically (e.g., a plugin system), you might not have a compile-time reference to the attribute type. You fall back to untyped reflection:

```csharp
Assembly asm = Assembly.LoadFrom("MyPlugin.dll");
Type t = asm.GetType("MyPlugin.MyClass");

// Get all attributes -- returns object[], can't cast to a known type
object[] attrs = t.GetCustomAttributes(false); // false = don't search base types

foreach (object attr in attrs)
{
    // Use reflection on the attribute itself to read its properties
    Type attrType = attr.GetType();
    PropertyInfo textProp = attrType.GetProperty("Text");
    if (textProp != null)
    {
        string value = (string)textProp.GetValue(attr);
        Console.WriteLine($"Attribute: {value}");
    }
}
```

```ad-info
The `bool inherit` parameter on `GetCustomAttributes(bool)` controls whether attributes from base classes are included. Pass `true` to walk the inheritance chain, `false` for only the declaring type.
```

## Early Binding vs Late Binding

| | Early Binding | Late Binding |
| --- | --- | --- |
| Type known at | Compile time | Runtime |
| Type safety | Strong -- compiler checks | None -- runtime errors |
| Syntax | `GetCustomAttribute<T>()` | `GetCustomAttributes()` + reflection |
| Performance | Faster | Slower (extra reflection layer) |
| Use when | You own or reference the attribute type | Plugin/extension systems where you can't reference the attribute assembly |

```ad-note
If you control both the host and plugins, prefer the **shared interface + early binding** approach. Define the attribute in a common library that both sides reference. Late binding is only necessary when you truly cannot share a type reference.
```

## Useful Methods for Attribute Reflection

| Method | Returns | Available on |
| --- | --- | --- |
| `GetCustomAttribute<T>()` | Single attribute or `null` | `MemberInfo`, `Type`, `Assembly` |
| `GetCustomAttributes<T>()` | All matching attributes | `MemberInfo`, `Type`, `Assembly` |
| `GetCustomAttributes(bool)` | All attributes as `object[]` | `MemberInfo` (bool = inherit) |
| `IsDefined(Type)` | `true` if attribute is applied | `MemberInfo`, `Type`, `Assembly` |
| `Attribute.GetCustomAttribute()` | Static helper (single) | `Attribute` class |

## See Also

- [[Custom Attributes]]
- [[AttributeUsage]]
- [[Reflection Overview]]
- [[Reflection Usage]]
- [[Early Binding and Late Binding]]
