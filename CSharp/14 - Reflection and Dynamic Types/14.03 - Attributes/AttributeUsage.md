---
tags:
 - csharp
 - attributes
---

## The `[AttributeUsage]` Attribute

`AttributeUsage` is applied to custom attribute classes to control where and how they can be used. It has three properties:

```csharp
[AttributeUsage(
    AttributeTargets.Class | AttributeTargets.Method,  // WHERE it can be applied
    AllowMultiple = true,                               // can apply more than once?
    Inherited = false                                    // inherited by derived classes?
)]
public class MyAttribute : Attribute { }
```

---

## AttributeTargets

Controls which code elements the attribute can decorate. Common values:

| Target                          | Applies to        |
| ------------------------------- | ----------------- |
| `AttributeTargets.Class`       | Classes           |
| `AttributeTargets.Method`      | Methods           |
| `AttributeTargets.Property`    | Properties        |
| `AttributeTargets.Field`       | Fields            |
| `AttributeTargets.Parameter`   | Method parameters |
| `AttributeTargets.ReturnValue` | Return values     |
| `AttributeTargets.Assembly`    | The entire assembly |
| `AttributeTargets.Interface`   | Interfaces        |
| `AttributeTargets.Struct`      | Structs           |
| `AttributeTargets.Enum`        | Enums             |
| `AttributeTargets.Constructor` | Constructors      |
| `AttributeTargets.All`         | Everything (default) |

Combine with bitwise OR:

```csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Struct | AttributeTargets.Interface)]
```

---

## AllowMultiple

Controls whether the attribute can be applied more than once to the same element. Default is `false`.

```csharp
// AllowMultiple = false (default) — compiler error if applied twice
[MyAttr("first")]
[MyAttr("second")]  // CS0579: Duplicate attribute
public class Foo { }

// AllowMultiple = true — both are applied
[MyAttr("first")]
[MyAttr("second")]  // OK
public class Foo { }
```

See [[AllowMultiple]] for detailed examples.

---

## Inherited

Controls whether a derived class inherits the attribute from its base class. Default is `true`.

```csharp
[AttributeUsage(AttributeTargets.Class, Inherited = true)]
public class TrackableAttribute : Attribute { }

[Trackable]
public class Base { }

public class Derived : Base { }

// With Inherited = true:
typeof(Derived).GetCustomAttributes(typeof(TrackableAttribute), inherit: true)
// Returns the TrackableAttribute — inherited from Base

// With Inherited = false:
// Returns nothing — attribute not inherited
```

```ad-note
The `inherit` parameter on `GetCustomAttributes` must be `true` for inheritance to work. Even with `Inherited = true` on the attribute, passing `inherit: false` to `GetCustomAttributes` skips inherited attributes.
```

---

## Defaults

| Property            | Default                | Meaning                     |
| ------------------- | ---------------------- | --------------------------- |
| ValidOn (targets)   | `AttributeTargets.All` | Can be applied anywhere     |
| AllowMultiple       | `false`                | Only once per element       |
| Inherited           | `true`                 | Derived classes inherit it  |

---

## See Also

- [[AllowMultiple]]
- [[Custom Attributes]]
- [[Attribute Overview]]
