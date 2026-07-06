---
tags:
 - csharp
 - reflection
 - assemblies
---

## Why Dynamically Load Assemblies?

When you add a project reference or NuGet package, the assembly is loaded automatically at startup -- that's **static loading**. Dynamic loading is for when you don't know at compile time which assemblies you'll need:

- **Plugin/extension systems** -- users drop DLLs into a folder, your app discovers them
- **Modular applications** -- load features on demand to reduce startup time
- **Tools that inspect arbitrary DLLs** -- analyzers, documentation generators, test runners

## Loading an Assembly at Runtime

Three main methods on the `Assembly` class:

```csharp
// By file path -- loads from disk
Assembly asm1 = Assembly.LoadFrom(@"C:\plugins\MyPlugin.dll");

// By assembly name -- uses standard probing rules (GAC, app base, etc.)
Assembly asm2 = Assembly.Load("MyPlugin");

// Isolated load -- no binding context, avoids version conflicts
Assembly asm3 = Assembly.LoadFile(@"C:\plugins\MyPlugin.dll");
```

| Method | Resolves from | Loads into | Use when |
| --- | --- | --- | --- |
| `LoadFrom(path)` | Exact file path | Default context | You know the file path |
| `Load(name)` | Probing rules (GAC, app base, etc.) | Default context | You know the assembly name |
| `LoadFile(path)` | Exact file path | No context (isolated) | Inspecting, avoiding conflicts |

```ad-note
`Assembly.ReflectionOnlyLoadFrom` existed in .NET Framework for inspect-without-execute scenarios, but it is **not available in .NET Core/.NET 5+**. Use `MetadataLoadContext` from the `System.Reflection.MetadataLoadContext` NuGet package instead.
```

## The Complete Pattern: Load, Get Type, Create Instance, Invoke

This is the fundamental workflow for dynamic assembly usage:

```csharp
// 1. Load the assembly
Assembly asm = Assembly.LoadFrom("MyPlugin.dll");

// 2. Get the type by its fully qualified name
Type pluginType = asm.GetType("MyPlugin.MyClass");

// 3. Create an instance via its default constructor
object instance = Activator.CreateInstance(pluginType);

// 4. Get and invoke a method
MethodInfo method = pluginType.GetMethod("Execute");
method.Invoke(instance, null);
```

## Working with Interfaces (Plugin Pattern)

The practical approach: define a **shared interface** in a common library, then load plugins that implement it. This gives you type safety on the host side while keeping plugins decoupled.

```csharp
// Shared interface (in a common library both host and plugins reference)
public interface IPlugin
{
    string Name { get; }
    void Execute();
}

// Host: load all plugins from a directory
string pluginDir = @"C:\plugins";
foreach (string dll in Directory.GetFiles(pluginDir, "*.dll"))
{
    Assembly asm = Assembly.LoadFrom(dll);

    foreach (Type t in asm.GetTypes())
    {
        if (typeof(IPlugin).IsAssignableFrom(t) && !t.IsAbstract)
        {
            IPlugin plugin = (IPlugin)Activator.CreateInstance(t);
            Console.WriteLine($"Loaded: {plugin.Name}");
            plugin.Execute();
        }
    }
}
```

This is how many real frameworks work -- ASP.NET middleware discovery, Visual Studio extensions, game mod systems, and MEF-based composition all follow this pattern.

```ad-info
The key insight is that `typeof(IPlugin).IsAssignableFrom(t)` checks whether `t` implements the interface, while `!t.IsAbstract` filters out the interface itself and any abstract base classes.
```

## Error Handling

Common exceptions when dynamically loading assemblies:

| Exception                   | Cause                                                       |
| --------------------------- | ----------------------------------------------------------- |
| `FileNotFoundException`     | Assembly DLL not found at the given path                    |
| `BadImageFormatException`   | File is not a valid .NET assembly (or wrong bitness)        | 
| `TypeLoadException`         | Type not found in the assembly                              |
| `MissingMethodException`    | Method doesn't exist or constructor signature doesn't match |
| `TargetInvocationException` | The invoked method itself threw -- check `.InnerException`  |

```ad-warning
`TargetInvocationException` wraps the real exception. Always inspect `.InnerException` to find the actual error. If you only log the outer exception, you'll miss the root cause.
```

## See Also

- [[Reflection Overview]]
- [[Reflection Usage]]
- [[Activator Class]]
- [[Building extendable application]]
