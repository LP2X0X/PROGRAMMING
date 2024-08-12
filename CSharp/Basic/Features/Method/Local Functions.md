Local functions in C# allow you to define methods within the scope of another method. They provide a way to encapsulate logic within a method without polluting the containing type's API. Local functions have access to the variables and parameters of the containing method, which can make them quite powerful.

Here's a basic example to illustrate local functions:

```csharp
public class Calculator
{
    public int Add(int a, int b)
    {
        // Local function to validate input
        bool ValidateInput()
        {
            return a >= 0 && b >= 0;
        }

        if (!ValidateInput())
        {
            throw new ArgumentException("Both numbers must be non-negative.");
        }

        // Local function to perform addition
        int PerformAddition()
        {
            return a + b;
        }

        return PerformAddition();
    }
}
```

In this example:
- Inside the `Add` method of the `Calculator` class, there are two local functions: `ValidateInput` and `PerformAddition`.
- These local functions are defined within the scope of the `Add` method and can access its parameters `a` and `b`.
- `ValidateInput` checks if both input numbers are non-negative. If not, it throws an exception.
- `PerformAddition` performs the actual addition of the two numbers.

Local functions can be beneficial for encapsulating logic that is specific to a particular method and doesn't need to be exposed outside of it. They can improve code readability and maintainability by keeping related logic together.

Keep in mind that local functions are only accessible within the method in which they are defined. They cannot be called from outside the containing method.