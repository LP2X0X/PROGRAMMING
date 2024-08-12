Dynamic loading in C# allows you to load assemblies and types at runtime, enabling you to work with types and members that are not known at compile time. This is useful for creating plugins, extensions, or applications that need to interact with assemblies dynamically.

Here’s how you can dynamically load an assembly, get a type from it, and create an instance of that type using reflection:

### Step-by-Step Example

1. **Create the Assembly to Load**:
   - First, create a separate assembly (DLL) that contains the types and methods you want to load dynamically.

```csharp
// Save this as MyLibrary.cs and compile it into a DLL
namespace MyLibrary
{
    public class MyClass
    {
        public void MyMethod()
        {
            Console.WriteLine("Hello from MyLibrary.MyClass.MyMethod!");
        }
    }
}
```

Compile this file into a DLL:

```sh
csc /target:library /out:MyLibrary.dll MyLibrary.cs
```

2. **Load the Assembly Dynamically**:
   - Create a new application that will load the compiled DLL at runtime and use its types and methods.

```csharp
using System;
using System.Reflection;

namespace DynamicLoadExample
{
    class Program
    {
        static void Main(string[] args)
        {
            // Path to the DLL
            string path = "MyLibrary.dll";

            // Load the assembly
            Assembly assembly = Assembly.LoadFrom(path);

            // Get the type
            Type myClassType = assembly.GetType("MyLibrary.MyClass");

            if (myClassType != null)
            {
                // Create an instance of the type
                object myClassInstance = Activator.CreateInstance(myClassType);

                // Get the method
                MethodInfo myMethod = myClassType.GetMethod("MyMethod");

                if (myMethod != null)
                {
                    // Invoke the method
                    myMethod.Invoke(myClassInstance, null);
                }
            }
        }
    }
}
```

### Explanation

1. **Assembly.LoadFrom**: This method loads an assembly given its file path. The assembly is loaded into the application domain, and you can then use reflection to interact with its types and members.

2. **Type.GetType**: This method gets a `Type` object representing the type with the specified name. In this case, it's `MyLibrary.MyClass`.

3. **Activator.CreateInstance**: This method creates an instance of the specified type. It uses the default constructor to create the instance.

4. **MethodInfo.GetMethod**: This method retrieves a `MethodInfo` object representing the method with the specified name. In this case, it's `MyMethod`.

5. **MethodInfo.Invoke**: This method invokes the specified method on the given object instance.

### Handling Errors and Edge Cases

When dynamically loading assemblies, it’s important to handle potential errors and edge cases:

- **FileNotFoundException**: Handle the case where the assembly file is not found.
- **TypeLoadException**: Handle the case where the specified type is not found in the assembly.
- **MissingMethodException**: Handle the case where the specified method is not found on the type.
- **TargetInvocationException**: Handle exceptions thrown by the invoked method.

### Enhanced Example with Error Handling

```csharp
using System;
using System.Reflection;

namespace DynamicLoadExample
{
    class Program
    {
        static void Main(string[] args)
        {
            // Path to the DLL
            string path = "MyLibrary.dll";

            try
            {
                // Load the assembly
                Assembly assembly = Assembly.LoadFrom(path);

                // Get the type
                Type myClassType = assembly.GetType("MyLibrary.MyClass");

                if (myClassType != null)
                {
                    // Create an instance of the type
                    object myClassInstance = Activator.CreateInstance(myClassType);

                    // Get the method
                    MethodInfo myMethod = myClassType.GetMethod("MyMethod");

                    if (myMethod != null)
                    {
                        // Invoke the method
                        myMethod.Invoke(myClassInstance, null);
                    }
                    else
                    {
                        Console.WriteLine("Method 'MyMethod' not found in type 'MyLibrary.MyClass'.");
                    }
                }
                else
                {
                    Console.WriteLine("Type 'MyLibrary.MyClass' not found in assembly.");
                }
            }
            catch (FileNotFoundException ex)
            {
                Console.WriteLine("Assembly file not found: " + ex.Message);
            }
            catch (TypeLoadException ex)
            {
                Console.WriteLine("Type not found: " + ex.Message);
            }
            catch (MissingMethodException ex)
            {
                Console.WriteLine("Method not found: " + ex.Message);
            }
            catch (TargetInvocationException ex)
            {
                Console.WriteLine("Error invoking method: " + ex.Message);
            }
            catch (Exception ex)
            {
                Console.WriteLine("An error occurred: " + ex.Message);
            }
        }
    }
}
```

This enhanced example includes basic error handling to manage common issues that can arise when dynamically loading assemblies and invoking methods. This approach ensures that your application can handle errors gracefully and provide meaningful messages to the user.