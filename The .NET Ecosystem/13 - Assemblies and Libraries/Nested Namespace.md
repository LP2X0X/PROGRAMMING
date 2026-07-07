---
tags:
  - csharp
  - namespaces
---

## Namespaces and Folder Structure

```ad-warning
Directory structure has **no impact** on namespaces in C#. A class in `Foo/Bar/Baz.cs` can declare any namespace it wants — the compiler does not enforce a match between file path and namespace.
```

That said, the **strong convention** is to match them. Reasons this matters in practice:

- **IDE tooling** — Visual Studio and Rider auto-generate namespaces from folder paths when creating new files.
- **Analyzers** — the `IDE0130` diagnostic (namespace does not match folder structure) fires by default in modern .NET projects.
- **Developer navigation** — when namespaces mirror folders, you can find the file for a type by mentally mapping `Company.Product.Feature` to `Company/Product/Feature/`.
- **Template-generated code** — `dotnet new` scaffolding relies on folder-to-namespace mapping.

Breaking the convention is legal but creates friction. Follow it unless you have a specific reason not to.

---

## Nesting Styles

C# offers three ways to express nested namespaces. All three produce identical IL — the choice is purely syntactic.

### Option 1: Traditional Nested Blocks

```csharp
namespace CustomNamespaces
{
    namespace MyShapes
    {
        public class Circle
        {
            // Fully qualified: CustomNamespaces.MyShapes.Circle
        }
    }
}
```

Each nesting level adds a pair of braces and an indentation level. This was the only option before C# 10.

### Option 2: File-Scoped Root + Nested Block

```csharp
namespace CustomNamespaces;     // file-scoped (C# 10+)
namespace MyShapes              // block-scoped nested inside
{
    public class Circle
    {
        // Fully qualified: CustomNamespaces.MyShapes.Circle
    }
}
```

The file-scoped declaration removes one nesting level for the outermost namespace while keeping block syntax for the inner one.

### Option 3: Fully File-Scoped with Dot Notation (Modern Default)

```csharp
namespace CustomNamespaces.MyShapes;    // file-scoped, C# 10+

public class Circle
{
    // Fully qualified: CustomNamespaces.MyShapes.Circle
}
```

The entire namespace hierarchy is expressed in a single line. All types in the file belong to that namespace with zero extra indentation.

---

### Comparison Table

| Aspect                  | Option 1 (Nested Blocks) | Option 2 (Mixed)           | Option 3 (File-Scoped Dot) |
| ----------------------- | ------------------------ | -------------------------- | -------------------------- |
| C# version required     | Any                      | C# 10+                     | C# 10+                     |
| Indentation levels      | N (one per nesting)      | N - 1                      | 0                           |
| Multiple namespaces/file| Yes                      | Yes (one file-scoped max)  | No — one namespace per file |
| Modern .NET default     | No                       | No                         | **Yes**                     |
| Best for                | Legacy code, rare multi-namespace files | Transitional codebases | New projects, single-namespace files |

```ad-note
The .NET SDK template default since .NET 6 is **Option 3** (file-scoped namespaces). The `dotnet format` tool and `IDE0161` analyzer both encourage it. If your team hasn't adopted it yet, a bulk conversion is a single `dotnet format` run with the right `.editorconfig` setting:

```
[*.cs]
csharp_style_namespace_declarations = file_scoped
```
```

---

## See Also

- [[Fully Qualified Name]]
- [[Aliases]]
- [[The Role of .NET Assemblies]]
