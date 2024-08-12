In C#, access modifiers are keywords used to specify the accessibility of types and members (such as classes, methods, and variables). They define how the members of a class or the class itself can be accessed in the code. Here are the main access modifiers in C#:

1. **public**: 
   - The member or type is accessible from any other code in the same assembly or another assembly that references it.
   - Example: 
     ```csharp
     public class MyClass
     {
         public int MyProperty { get; set; }
     }
     ```

2. **private**: 
   - The member is accessible only within the body of the class or the struct in which it is declared.
   - Example: 
     ```csharp
     class MyClass
     {
         private int myField;
     }
     ```

3. **protected**: 
   - The member is accessible within its class and by derived class instances.
   - Example: 
     ```csharp
     class MyBaseClass
     {
         protected int myProtectedField;
     }
     
     class MyDerivedClass : MyBaseClass
     {
         void MyMethod()
         {
             myProtectedField = 10;
         }
     }
     ```

4. **internal**: 
   - The member or type is accessible only within files in the same assembly.
   - Example: 
     ```csharp
     internal class MyClass
     {
         internal int myField;
     }
     ```

5. **protected internal**: 
   - The member is accessible from the current assembly or from types that are derived from the containing class, even if they are in a different assembly.
   - Example: 
     ```csharp
     class MyClass
     {
         protected internal int myField;
     }
     ```

6. **private protected**: 
   - The member is accessible only within its containing class or types derived from the containing class within the same assembly.
   - Example: 
     ```csharp
     class MyClass
     {
         private protected int myField;
     }
     ```

### Example Class with Different Access Modifiers

```csharp
public class ExampleClass
{
    // Public field - accessible from anywhere
    public int publicField;

    // Private field - accessible only within this class
    private int privateField;

    // Protected field - accessible within this class and derived classes
    protected int protectedField;

    // Internal field - accessible within the same assembly
    internal int internalField;

    // Protected Internal field - accessible within the same assembly or derived classes
    protected internal int protectedInternalField;

    // Private Protected field - accessible within this class or derived classes in the same assembly
    private protected int privateProtectedField;

    public void PublicMethod()
    {
        // All fields are accessible here
        privateField = 1;
        protectedField = 2;
        internalField = 3;
        protectedInternalField = 4;
        privateProtectedField = 5;
    }
}
```

Understanding and properly using access modifiers is crucial for encapsulation and maintaining the integrity of your code. They help in defining clear boundaries for how data and functionality can be accessed and modified.