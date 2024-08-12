In the context of .NET and C#, late binding generally refers to the process of resolving method calls at runtime rather than at compile time. This is achieved through mechanisms such as reflection or dynamic typing. Here’s a brief overview of each:

1. **Reflection:** With reflection, you can inspect and invoke types and members dynamically at runtime. This allows you to discover and call methods, properties, and fields on objects without necessarily knowing them at compile time. Reflection can be powerful but is often slower and less type-safe compared to static binding.

2. **Dynamic:** C# introduced the `dynamic` keyword, which allows for late binding through dynamic typing. When a variable is declared as `dynamic`, the C# compiler defers type checking and member resolution until runtime. This can be useful when working with COM interop, dynamic languages, or scenarios where the type is determined at runtime.

Here’s a simple example using reflection in C#:

```csharp
using System;
using System.Reflection;

public class Program
{
    public static void Main()
    {
        // Example of late binding using reflection
        Type type = typeof(Math);
        MethodInfo method = type.GetMethod("Pow", new[] { typeof(double), typeof(double) });
        if (method != null)
        {
            double result = (double)method.Invoke(null, new object[] { 2, 3 });
            Console.WriteLine("Result: " + result);
        }
    }
}
```

In this example:
- We obtain a reference to the `Pow` method of the `Math` class at runtime using `GetMethod` from `Type`.
- We then invoke this method dynamically using `Invoke`.

This approach allows for flexibility but requires careful handling due to potential performance overhead and lack of compile-time safety.