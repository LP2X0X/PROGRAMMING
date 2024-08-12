When using lambda expressions in C#, the concepts of `static` and `discard` have specific meanings and usage scenarios:

### Static in Lambda Expressions

Since lambda expressions are shorthand for delegates, it is understandable that lambda also support the static keyword (with C# 9.0) as well as discards (covered in the next section). When using the static keyword, lambda expression can not update a variable declared in an outer scope.

```csharp
var outerVariable = 0;  

Func<int, int, bool> DoWork = static (x,y) =>  
{  
	//Compile error since it’s accessing an outer variable  
	//outerVariable++;  
	return true;  
};
```

### Discard in Lambda Expressions

Discards (`_`) in lambda expressions can be used in scenarios where you want to ignore certain parameters or parts of a tuple. They are particularly useful when you don't need to use all parameters passed to the lambda expression.

```csharp
public class ExampleClass
{
    public void UseLambda()
    {
        Action<int, string> logInfo = (_, message) =>
        {
            Console.WriteLine(message);
        };

        logInfo(1, "Logging message"); // Output: Logging message
    }
}
```

In this example, `_` is used as a discard for the first parameter of the `logInfo` lambda expression, indicating that the first parameter (an integer) is ignored.

### Combining Static and Discard

When using lambda expressions, you can combine `static` variables or methods with discards if needed, depending on your use case. Here’s an example that demonstrates accessing a `static` variable and using a discard:

```csharp
public class ExampleClass
{
    private static int staticValue = 10;

    public void UseLambda()
    {
        Func<int, int> addStatic = (_, y) => staticValue + y;

        Console.WriteLine(addStatic(0, 5)); // Output: 15
    }
}
```

In this example:
- `_` is used as a discard for the first parameter of the `addStatic` lambda expression.
- The lambda expression accesses the `staticValue` and adds it to the second parameter `y`.

### Summary

- **Static**: Lambda expressions can access `static` members of their containing class directly.
- **Discard**: Lambda expressions can use `_` as a discard for parameters they don't need to use.

Understanding these features allows you to leverage lambda expressions effectively in C# for concise and expressive code, while still accessing necessary context through `static` members or ignoring unnecessary parameters using discards.