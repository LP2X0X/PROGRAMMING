---
tags:
 - csharp
 - reflection
 - metadata
---

## The Field Metadata Token

The **Field token** describes each field defined in a type — instance fields, static fields, constants, and enum members. Each gets a `Field #n` entry in the metadata.

### Regular Class Fields

```csharp
public class Person
{
    private string _name;
    public int Age;
    public static int Count;
}
```

In IL metadata:

```il
.field private string _name
.field public int32 Age
.field public static int32 Count
```

Each Field entry records:

- **Name** — the field identifier
- **Type** — the field's data type (mapped to IL type names like `int32`, `string`, etc.)
- **Accessibility** — `public`, `private`, `family` (protected), `assembly` (internal), etc.
- **Static or instance** — whether the `static` modifier is present
- **Other modifiers** — `initonly` (readonly), `literal` (const), `volatile`, etc.

```ad-note
C# properties do **not** appear as Field tokens. A property compiles into a backing field (if auto-implemented) plus getter/setter methods. The backing field shows up as a Field token with a compiler-generated name like `<Name>k__BackingField`, while the getter/setter appear as Method tokens.
```

### Enum Fields

Enums are a special case. Each named member becomes a Field token with a `static literal` value, plus a hidden instance field that holds the underlying integer:

```csharp
public enum Color { Red, Green, Blue }
```

```il
.field public specialname rtspecialname int32 value__    // hidden backing field
.field public static literal valuetype Color Red = int32(0)
.field public static literal valuetype Color Green = int32(1)
.field public static literal valuetype Color Blue = int32(2)
```

- `value__` — the hidden instance field that stores the actual integer value at runtime
- Each named member (`Red`, `Green`, `Blue`) — a `static literal` field with a compile-time constant value

### Const vs Readonly in Metadata

| C# Declaration | IL Modifier | Behavior |
|---|---|---|
| `const int X = 5;` | `static literal int32 X = int32(5)` | Value baked in at compile time |
| `readonly int Y;` | `initonly int32 Y` | Value assigned at runtime (constructor only) |
| `static readonly int Z;` | `static initonly int32 Z` | Value assigned in static constructor |

```ad-warning
Because `const` fields are baked into the metadata as literal values, changing a `const` in a library requires recompiling all assemblies that reference it. The consuming assembly copied the literal value into its own IL — it does not re-read the original at runtime. Use `static readonly` instead if the value may change between versions.
```

## See Also

- [[TypeDef and TypeRef]]
- [[Extends]]
- [[User Strings]]
