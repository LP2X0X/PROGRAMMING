In .NET Core and .NET 5+, `AssemblyLoadContext` is used for assembly isolation, allowing dynamic loading and unloading of assemblies. This mechanism replaces the application domains (`AppDomain`) used in the .NET Framework. `AssemblyLoadContext` provides more flexibility and control over assembly loading behavior, enabling scenarios such as plugin architectures and hot-reloading of assemblies.

### Key Concepts of AssemblyLoadContext

1. **Isolation**: Assemblies loaded in different `AssemblyLoadContext` instances are isolated from each other.
2. **Unloading**: Assemblies loaded in a custom `AssemblyLoadContext` can be unloaded, allowing for better memory management and modularity.
3. **Custom Load Contexts**: Developers can create custom load contexts to control the loading behavior of assemblies.

### Basic Example of Using AssemblyLoadContext

Here’s a simple example demonstrating how to load and unload an assembly using `AssemblyLoadContext`.

#### Creating and Using AssemblyLoadContext

```csharp
using System;
using System.Reflection;
using System.Runtime.Loader;

class Program
{
    static void Main()
    {
        // Create a new custom AssemblyLoadContext
        var loadContext = new CustomAssemblyLoadContext();

        // Load an assembly into the custom load context
        var assemblyPath = "path\\to\\your\\assembly.dll";
        var assembly = loadContext.LoadFromAssemblyPath(assemblyPath);

        // Get the type and method to execute
        Type type = assembly.GetType("Namespace.ClassName");
        MethodInfo method = type.GetMethod("MethodName");

        // Execute the method
        method.Invoke(null, null);

        // Unload the custom load context
        loadContext.Unload();

        // Force garbage collection to unload the assembly
        for (int i = 0; i < 10; i++)
        {
            GC.Collect();
            GC.WaitForPendingFinalizers();
        }

        Console.WriteLine("Assembly unloaded");
    }
}

// Custom AssemblyLoadContext class
class CustomAssemblyLoadContext : AssemblyLoadContext
{
    protected override Assembly Load(AssemblyName assemblyName)
    {
        // Custom logic to load assemblies, if needed
        return null;
    }
}
```

### Detailed Example with Plugin Architecture

In this example, we’ll create a plugin architecture where plugins are loaded into separate `AssemblyLoadContext` instances, allowing them to be unloaded independently.

#### Plugin Interface

First, define a common interface for plugins:

```csharp
public interface IPlugin
{
    void Execute();
}
```

#### Plugin Implementation

Next, create a simple plugin implementation:

```csharp
public class Plugin : IPlugin
{
    public void Execute()
    {
        Console.WriteLine("Plugin executed.");
    }
}
```

#### Plugin Loader

Create a class to load and execute plugins:

```csharp
using System;
using System.IO;
using System.Reflection;
using System.Runtime.Loader;

public class PluginLoader : AssemblyLoadContext
{
    private AssemblyDependencyResolver _resolver;

    public PluginLoader(string pluginPath) : base(true)
    {
        _resolver = new AssemblyDependencyResolver(pluginPath);
    }

    protected override Assembly Load(AssemblyName assemblyName)
    {
        string assemblyPath = _resolver.ResolveAssemblyToPath(assemblyName);
        if (assemblyPath != null)
        {
            return LoadFromAssemblyPath(assemblyPath);
        }
        return null;
    }

    public void ExecutePlugin(string pluginAssemblyPath)
    {
        var assembly = LoadFromAssemblyPath(pluginAssemblyPath);
        var type = assembly.GetType("Namespace.Plugin");
        var plugin = Activator.CreateInstance(type) as IPlugin;
        plugin.Execute();
    }
}
```

#### Main Application

Finally, create the main application to load and execute plugins:

```csharp
using System;

class Program
{
    static void Main()
    {
        string pluginPath = "path\\to\\your\\plugin.dll";

        var loader = new PluginLoader(pluginPath);
        loader.ExecutePlugin(pluginPath);

        loader.Unload();

        // Force garbage collection to unload the plugin
        for (int i = 0; i < 10; i++)
        {
            GC.Collect();
            GC.WaitForPendingFinalizers();
        }

        Console.WriteLine("Plugin unloaded");
    }
}
```

### Summary

`AssemblyLoadContext` in .NET Core and .NET 5+ offers a robust mechanism for isolating and managing assemblies. This approach is especially useful for creating plugin systems, where plugins can be dynamically loaded and unloaded, ensuring better memory management and application modularity. The examples provided illustrate basic and advanced usage scenarios, enabling you to leverage `AssemblyLoadContext` for flexible and efficient assembly loading.