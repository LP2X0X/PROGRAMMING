---
tags:
 - csharp
 - exception-handling
---

## `Exception.TargetSite`

The `TargetSite` property returns a `System.Reflection.MethodBase` object describing the method that threw the exception. It gives you structured, programmatic access to the method's name, return type, parameters, and declaring class -- far richer than parsing the `StackTrace` string.

### Basic Usage

```csharp
try
{
    car.Accelerate(100);
}
catch (Exception ex)
{
    Console.WriteLine($"Method:    {ex.TargetSite}");
    Console.WriteLine($"Class:     {ex.TargetSite.DeclaringType}");
    Console.WriteLine($"Member:    {ex.TargetSite.MemberType}");
    Console.WriteLine($"Message:   {ex.Message}");
    Console.WriteLine($"Source:    {ex.Source}");
}

// Output:
//   Method:    Void Accelerate(Int32)
//   Class:     MyApp.Car
//   Member:    Method
//   Message:   Car has overheated!
//   Source:    MyApp
```

### Exploring the `MethodBase` Object

`TargetSite` returns a `MethodBase`, which opens up the full Reflection API:

```csharp
catch (Exception ex)
{
    MethodBase? method = ex.TargetSite;

    if (method is not null)
    {
        Console.WriteLine($"Method name:    {method.Name}");
        Console.WriteLine($"Declaring type: {method.DeclaringType?.FullName}");
        Console.WriteLine($"Is public:      {method.IsPublic}");
        Console.WriteLine($"Is static:      {method.IsStatic}");
        Console.WriteLine($"Return type:    {(method as MethodInfo)?.ReturnType.Name}");

        Console.WriteLine("Parameters:");
        foreach (var param in method.GetParameters())
        {
            Console.WriteLine($"  {param.ParameterType.Name} {param.Name}");
        }
    }
}
```

### Tips & Best Practices

- **`TargetSite` can be `null`.** This happens when the runtime doesn't have metadata available (e.g., in certain partial-trust or dynamically generated code scenarios). Always null-check before accessing its members.
- **Printing `TargetSite` directly gives a method signature string** like `Void Accelerate(Int32)`. This is the `ToString()` of `MethodBase`, which includes return type, method name, and parameter types.
- **Use `DeclaringType` to identify the class.** `ex.TargetSite.DeclaringType.FullName` gives the fully qualified class name (namespace + class), which is especially helpful when multiple classes have methods with the same name.
- **`MemberType` tells you what kind of member threw.** It returns a `MemberTypes` enum value (`Method`, `Constructor`, `Property`, etc.), which helps distinguish between exceptions thrown from getters/setters vs regular methods.
- **Use for diagnostics, not control flow.** Inspecting `TargetSite` via reflection is useful for logging and debugging, but branching logic based on it creates fragile code that breaks when methods are renamed.
- **`Source` is a related property worth pairing.** `ex.Source` returns the assembly name where the exception originated. Together with `TargetSite`, you get assembly + class + method without parsing strings.
- **Prefer structured logging over manual inspection.** Most logging frameworks (Serilog, NLog) automatically capture `TargetSite` details when you log an exception object. Use `logger.Error(ex, "...")` rather than manually extracting these fields.
