A lambda expression in C# is a concise way to define anonymous functions or delegates inline. Lambda expressions were introduced in C# 3.0 and have become a fundamental feature of the language, especially in scenarios where you need to pass a method as an argument to another method or define simple, one-time-use functions without explicitly naming them.

```ad-note
Understand that lambda expressions can be used anywhere you would have used an anonymous method or a [[Strongly and Loosely Type|strongly type]] delegate
```

```ad-attention
Lambda expressions are simply anonymous methods in disguise (which greatly improve readability).
```
### Syntax

The syntax of a lambda expression consists of the following parts:

```csharp
(parameters) => expression_or_statement_block
```

- **Parameters**: Optional list of parameters enclosed in parentheses. If there's exactly one parameter and its type can be inferred, you can omit the parentheses around the parameter list.
  
- **Lambda Operator (`=>`)**: Separates the parameter list from the lambda body.
  
- **Expression or Statement Block**: The body of the lambda expression, which can be a single expression or a block of statements enclosed in curly braces `{ }`.

### Examples

#### Example 1: Simple Lambda Expression

```csharp
// Lambda expression that squares a number
Func<int, int> square = x => x * x;

Console.WriteLine(square(5)); // Output: 25
```

In this example:
- `x` is the parameter of the lambda expression.
- `x => x * x` defines a lambda expression that takes an `int` parameter `x` and returns `x * x`, which is the square of `x`.
- `Func<int, int>` declares a delegate type that takes an `int` parameter and returns an `int`.

#### Example 2: Lambda Expression with Multiple Parameters

```csharp
// Lambda expression with multiple parameters
Func<int, int, int> add = (a, b) => a + b;

Console.WriteLine(add(3, 4)); // Output: 7
```

In this example:
- `(a, b)` are the parameters of the lambda expression.
- `(a, b) => a + b` defines a lambda expression that takes two `int` parameters `a` and `b` and returns their sum `a + b`.
- `Func<int, int, int>` declares a delegate type that takes two `int` parameters and returns an `int`.

#### Example 3: Lambda Expression with Statement Block

```csharp
// Lambda expression with a statement block
Func<int, int> factorial = n =>
{
    int result = 1;
    for (int i = 2; i <= n; i++)
    {
        result *= i;
    }
    return result;
};

Console.WriteLine(factorial(5)); // Output: 120
```

In this example:
- `n` is the parameter of the lambda expression.
- The lambda body `{ ... }` contains multiple statements to calculate the factorial of `n`.
- `Func<int, int>` declares a delegate type that takes an `int` parameter and returns an `int`.

### Benefits of Lambda Expressions

- **Conciseness**: Lambda expressions reduce the amount of boilerplate code required compared to traditional anonymous methods.
  
- **Readability**: They make the code more readable by keeping related logic closer together.

- **Functional Programming**: Lambda expressions facilitate functional programming paradigms, such as passing functions as arguments to other functions (higher-order functions).

### Usage Scenarios

Lambda expressions are commonly used in:
- **LINQ Queries**: For defining projections, filters, and ordering criteria.
- **Event Handlers**: For subscribing to events using anonymous functions.
- **Parallel and Asynchronous Programming**: For defining tasks and actions inline.

### Conclusion

Lambda expressions in C# are a powerful feature that enhances code expressiveness and supports modern programming paradigms like functional programming. They provide a flexible way to define small, inline functions or delegates without the need for explicit method declarations, thereby improving code readability and maintainability.