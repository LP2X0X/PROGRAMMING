Custom attributes in C# allow you to define your own metadata that can be applied to types, methods, properties, fields, parameters, and other program elements. They provide a way to annotate your code with additional information that can be inspected at runtime using reflection. Here's how you can create and use custom attributes:

### Creating a Custom Attribute

To create a custom attribute, you define a class that inherits from `System.Attribute`. You can add properties to this class to hold the metadata you want to associate with the attributed code element. Here’s a basic example:

```csharp
using System;

// Define a custom attribute
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, AllowMultiple = true)]
public class MyCustomAttribute : Attribute
{
    public MyCustomAttribute(string description)
    {
        Description = description;
    }

    public string Description { get; }
}
```

In this example:
- `MyCustomAttribute` is defined as a custom attribute that inherits from `System.Attribute`.
- The `AttributeUsage` attribute specifies where this custom attribute can be applied (`AttributeTargets.Class | AttributeTargets.Method`) and whether it can be applied multiple times (`AllowMultiple = true`).
- The constructor takes a `description` parameter, which initializes the `Description` property of the attribute.

### Applying a Custom Attribute

You can apply your custom attribute to various elements in your code by placing it in square brackets (`[]`) before the code element:

```csharp
// Apply the custom attribute to a class
[MyCustom("This is a class with a custom attribute")]
public class MyClass
{
    // Apply the custom attribute to a method
    [MyCustom("This method has a custom attribute")]
    public void MyMethod()
    {
        // Method implementation
    }
}
```

### Retrieving Custom Attribute Information at Runtime

To retrieve information stored in a custom attribute at runtime, you can use reflection. Here’s an example of how you might retrieve and use the `MyCustomAttribute` applied to `MyClass` and `MyMethod`:

```csharp
using System;
using System.Reflection;

public class Program
{
    public static void Main()
    {
        // Reflectively inspect attributes applied to MyClass
        Type type = typeof(MyClass);
        var classAttributes = type.GetCustomAttributes(typeof(MyCustomAttribute), false);
        foreach (MyCustomAttribute attr in classAttributes)
        {
            Console.WriteLine($"Class attribute: {attr.Description}");
        }

        // Reflectively inspect attributes applied to MyMethod
        var method = type.GetMethod("MyMethod");
        var methodAttributes = method.GetCustomAttributes(typeof(MyCustomAttribute), false);
        foreach (MyCustomAttribute attr in methodAttributes)
        {
            Console.WriteLine($"Method attribute: {attr.Description}");
        }
    }
}
```

In this example:
- `typeof(MyCustomAttribute)` is used with `GetCustomAttributes` to retrieve all instances of `MyCustomAttribute` applied to `MyClass` and `MyMethod`.
- The retrieved attributes are then iterated over to access their properties (in this case, `Description`).

Custom attributes are powerful tools for adding metadata and behavior to your code, enabling you to annotate your code with custom information that can be used by frameworks, tools, or your own runtime logic.