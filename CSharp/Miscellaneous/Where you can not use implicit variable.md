In C#, implicit variables (`var` keyword) can be incredibly convenient for reducing verbosity and enhancing readability of code. However, there are specific scenarios where you cannot use implicit variables:

1. **Member Variables**: You cannot use `var` for member variables (fields) at the class level because C# requires explicit type declarations for fields.

   ```csharp
   // Incorrect usage
   // var myField = 10; // This would cause a compile-time error

   // Correct usage
   int myField = 10;
   ```

2. **Method Signatures**: You cannot use `var` for parameters or return types of methods. Method signatures require explicit type declarations.

   ```csharp
   // Incorrect usage
   // public var SomeMethod(var input) { ... }

   // Correct usage
   public int SomeMethod(int input) { ... }
   ```

3. **Static Variables**: Similar to member variables, static variables at the class level must have explicit type declarations.

   ```csharp
   // Incorrect usage
   // static var staticField = "Static";

   // Correct usage
   static string staticField = "Static";
   ```

4. **Array Initializers**: When initializing arrays using implicit array initialization syntax, you cannot use `var` alone without specifying the array type explicitly.

   ```csharp
   // Incorrect usage
   // var myArray = { 1, 2, 3 }; // This would cause a compile-time error

   // Correct usage
   int[] myArray = { 1, 2, 3 };
   ```

5. **Anonymous Types**: While you can use `var` to declare variables of anonymous types (`new { ... }`), you cannot declare an anonymous type member explicitly using `var`.

   ```csharp
   // Correct usage
   var person = new { Name = "Alice", Age = 30 };

   // Incorrect usage
   // var person = new { var Name = "Alice", var Age = 30 }; // This would cause a compile-time error
   ```

In summary, you cannot use implicit variables (`var`) in contexts where the type cannot be inferred from the initializer or where explicit type declarations are required by the C# language syntax (such as member variables, method signatures, static variables, array initializers, and anonymous type members). Using explicit type declarations ensures clarity and maintains the compiler's ability to enforce type safety effectively.