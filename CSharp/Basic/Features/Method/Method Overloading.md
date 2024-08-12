Overloading methods in C# allows you to define multiple methods with the same name but with different parameter lists within the same class or struct. Each method with the same name is said to be an "overload" of the method. The compiler determines which overload to call based on the number and types of arguments passed.

Here's an example to illustrate method overloading:

```csharp
public class Calculator
{
    // Method overload 1: Add method that takes two integers
    public int Add(int a, int b)
    {
        return a + b;
    }

    // Method overload 2: Add method that takes three integers
    public int Add(int a, int b, int c)
    {
        return a + b + c;
    }

    // Method overload 3: Add method that takes two doubles
    public double Add(double a, double b)
    {
        return a + b;
    }
}
```

In this example:
- We have defined three overloads of the `Add` method in the `Calculator` class.
- The first overload takes two integers, the second overload takes three integers, and the third overload takes two doubles.
- All three methods have the same name (`Add`), but they have different parameter lists.

When you call the `Add` method, the compiler determines which overload to call based on the arguments you provide. For example:

```csharp
Calculator calc = new Calculator();

int sum1 = calc.Add(1, 2);         // Calls the first overload
int sum2 = calc.Add(1, 2, 3);      // Calls the second overload
double sum3 = calc.Add(1.5, 2.5);  // Calls the third overload
```

Method overloading provides a way to create more readable and expressive APIs by allowing methods with the same purpose to be grouped together under the same name. It's important to note that method overloading is based on the number and types of parameters, not on the return type of the method.