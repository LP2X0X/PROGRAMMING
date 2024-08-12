The null-coalescing operator in C# is a useful feature for dealing with null values. It allows you to provide a default value in case a nullable type is null. There are two null-coalescing operators in C#:

1. `??` Operator
2. `??=` Operator

### `??` Operator

The `??` operator returns the left-hand operand if it is not null; otherwise, it returns the right-hand operand. This is particularly useful for providing default values for nullable types or reference types that might be null.

#### Syntax

```csharp
var result = nullableValue ?? defaultValue;
```

### Example Usage

Here's an example demonstrating how to use the `??` operator:

```csharp
using System;

class Program
{
    static void Main()
    {
        string? nullableString = null;
        string defaultString = "Default value";

        // Using the null-coalescing operator
        string result = nullableString ?? defaultString;

        Console.WriteLine(result); // Output: Default value

        // Another example with an integer
        int? nullableInt = null;
        int defaultInt = 10;

        int intResult = nullableInt ?? defaultInt;

        Console.WriteLine(intResult); // Output: 10

        // Example with non-null value
        nullableString = "Hello, World!";
        result = nullableString ?? defaultString;

        Console.WriteLine(result); // Output: Hello, World!
    }
}
```

### `??=` Operator

The `??=` operator is a compound assignment operator that assigns the right-hand operand to the left-hand operand only if the left-hand operand is null. This operator is available from C# 8.0 onwards.

#### Syntax

```csharp
nullableValue ??= defaultValue;
```

### Example Usage

Here's an example demonstrating how to use the `??=` operator:

```csharp
using System;

class Program
{
    static void Main()
    {
        string? nullableString = null;
        string defaultString = "Default value";

        // Using the null-coalescing assignment operator
        nullableString ??= defaultString;

        Console.WriteLine(nullableString); // Output: Default value

        // Another example with an integer
        int? nullableInt = null;
        int defaultInt = 10;

        nullableInt ??= defaultInt;

        Console.WriteLine(nullableInt); // Output: 10

        // Example with non-null value
        nullableString = "Hello, World!";
        nullableString ??= defaultString;

        Console.WriteLine(nullableString); // Output: Hello, World!
    }
}
```

### Key Points

- The `??` operator is used to return the left operand if it is not null; otherwise, it returns the right operand.
- The `??=` operator assigns the right operand to the left operand only if the left operand is null.
- These operators are especially useful when dealing with nullable value types and reference types to provide default values and prevent null reference exceptions.

Using the null-coalescing operators can make your code more concise and readable by reducing the need for explicit null checks.