---
tags: [csharp, asp-net-core, startup, program]
---


The minimal hosting model relies on C# 9's **top-level statements** feature. This allows a `.cs` file to contain executable code without wrapping it in a class or `Main` method.

### How It Works

```csharp
// This is a complete, valid C# program:
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();
app.MapGet("/", () => "Hello, World!");
app.Run();
```

The compiler generates an implicit `Program` class with a `Main` method behind the scenes.

> [!ad-note] The Hidden Program Class
> Even with top-level statements, a `Program` class exists at compile time. This matters for:
> - Integration testing: you reference `Program` as the entry point assembly.
> - Accessing the class in test projects: add `InternalsVisibleTo` or make `Program` partial.

```csharp
// At the bottom of Program.cs, add this for integration test access:
public partial class Program { }
```

### args Is Available

The `args` parameter (command-line arguments) is implicitly available in top-level statements. You do not need to declare it.

```csharp
// args is available without declaration
var builder = WebApplication.CreateBuilder(args);
Console.WriteLine($"Started with {args.Length} arguments");
```

> [!summary] Section Summary
> - Top-level statements eliminate the `class Program` and `static void Main` boilerplate.
> - The compiler generates an implicit `Program` class with a `Main` method.
> - `args` is implicitly available for command-line argument access.
> - Add `public partial class Program { }` for integration test compatibility.
