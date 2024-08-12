Custom type conversions in C# allow you to define how one type is converted to another type. This is particularly useful when you have custom classes and you want to control how instances of these classes are converted to other types. There are two main ways to implement custom type conversions in C#:

1. **Implicit Conversions**: These conversions are automatically applied by the compiler when needed.
2. **Explicit Conversions**: These conversions require a cast to be explicitly specified by the developer.

### Defining Custom Conversions

To define custom conversions, you need to use the `implicit` and `explicit` keywords in the class that you want to convert from. Here’s a basic example:

#### Example: Converting Between `Temperature` and `double`

Let's create a `Temperature` class that can be implicitly converted to and from a `double`.

```csharp
using System;

public class Temperature
{
    public double DegreesCelsius { get; }

    public Temperature(double degreesCelsius)
    {
        DegreesCelsius = degreesCelsius;
    }

    // Implicit conversion from double to Temperature
    public static implicit operator Temperature(double degreesCelsius)
    {
        return new Temperature(degreesCelsius);
    }

    // Implicit conversion from Temperature to double
    public static implicit operator double(Temperature temp)
    {
        return temp.DegreesCelsius;
    }

    // Explicit conversion from Temperature to string
    public static explicit operator string(Temperature temp)
    {
        return $"{temp.DegreesCelsius} °C";
    }

    public override string ToString()
    {
        return $"{DegreesCelsius} °C";
    }
}

class Program
{
    static void Main()
    {
        // Implicit conversion from double to Temperature
        Temperature temp1 = 36.5;
        Console.WriteLine(temp1); // Output: 36.5 °C

        // Implicit conversion from Temperature to double
        double degrees = temp1;
        Console.WriteLine(degrees); // Output: 36.5

        // Explicit conversion from Temperature to string
        string tempString = (string)temp1;
        Console.WriteLine(tempString); // Output: 36.5 °C
    }
}
```

### Key Points

1. **Implicit Conversion**: When defining an implicit conversion, use the `implicit` keyword. This conversion happens automatically when the target type is needed.
   ```csharp
   public static implicit operator Temperature(double degreesCelsius)
   {
       return new Temperature(degreesCelsius);
   }

   public static implicit operator double(Temperature temp)
   {
       return temp.DegreesCelsius;
   }
   ```

2. **Explicit Conversion**: When defining an explicit conversion, use the `explicit` keyword. This conversion must be explicitly cast by the developer.
   ```csharp
   public static explicit operator string(Temperature temp)
   {
       return $"{temp.DegreesCelsius} °C";
   }
   ```

3. **Type Safety**: Implicit conversions should be used sparingly and only when it is guaranteed to be safe. If there is a potential for data loss or an unexpected outcome, it is better to use explicit conversions to make the developer aware of the conversion.

### Practical Use Case

Custom conversions can be particularly useful in the following scenarios:
- Converting between different units of measurement (e.g., temperature, distance).
- Simplifying API usage by allowing direct assignment or operations with primitive types.
- Ensuring type safety while providing flexibility in how objects are used.

By implementing custom type conversions, you can enhance the usability and readability of your code, making it more intuitive and reducing the likelihood of errors.