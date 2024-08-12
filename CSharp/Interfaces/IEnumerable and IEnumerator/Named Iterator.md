A named iterator in C# refers to a method that returns an `IEnumerable<T>` or `IEnumerator<T>` and uses the `yield` keyword to generate a sequence of values. This pattern allows you to define custom iteration logic and can be used to create complex iterators with specific naming and behavior.

### Named Iterator Example

Let’s create a named iterator method that generates a sequence of Fibonacci numbers:

```csharp
using System;
using System.Collections.Generic;

public class FibonacciGenerator
{
    public static IEnumerable<int> GenerateFibonacciSequence(int count)
    {
        if (count < 0)
        {
            throw new ArgumentOutOfRangeException(nameof(count), "Count must be non-negative.");
        }

        int previous = 0;
        int current = 1;
        
        for (int i = 0; i < count; i++)
        {
            yield return previous;
            int next = previous + current;
            previous = current;
            current = next;
        }
    }
}

public class Program
{
    public static void Main()
    {
        foreach (int number in FibonacciGenerator.GenerateFibonacciSequence(10))
        {
            Console.WriteLine(number);
        }
    }
}
```

### Explanation

1. **Method Definition:**
   ```csharp
   public static IEnumerable<int> GenerateFibonacciSequence(int count)
   ```
   - This method is named `GenerateFibonacciSequence` and it returns an `IEnumerable<int>`, indicating it yields a sequence of integers.

2. **Parameter Validation:**
   ```csharp
   if (count < 0)
   {
       throw new ArgumentOutOfRangeException(nameof(count), "Count must be non-negative.");
   }
   ```
   - This guard clause checks if the input `count` is negative and throws an exception if it is.

3. **Fibonacci Logic:**
   ```csharp
   int previous = 0;
   int current = 1;
   
   for (int i = 0; i < count; i++)
   {
       yield return previous;
       int next = previous + current;
       previous = current;
       current = next;
   }
   ```
   - Initializes the first two Fibonacci numbers.
   - Uses a `for` loop to generate the sequence up to the specified `count`.
   - `yield return` is used to return each Fibonacci number one at a time.

### Usage in Main Method

The `Main` method demonstrates how to use the named iterator:

```csharp
foreach (int number in FibonacciGenerator.GenerateFibonacciSequence(10))
{
    Console.WriteLine(number);
}
```

- Calls the `GenerateFibonacciSequence` method with a count of 10.
- Iterates through the generated sequence and prints each Fibonacci number.

### More Complex Named Iterator Example

Here’s a more complex example where a named iterator filters and transforms a list of numbers:

```csharp
using System;
using System.Collections.Generic;

public static class NumberProcessor
{
    public static IEnumerable<int> FilterAndDoubleEvenNumbers(IEnumerable<int> numbers)
    {
        if (numbers == null)
        {
            throw new ArgumentNullException(nameof(numbers));
        }

        foreach (int number in numbers)
        {
            if (number % 2 == 0)
            {
                yield return number * 2;
            }
        }
    }
}

public class Program
{
    public static void Main()
    {
        List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

        foreach (int number in NumberProcessor.FilterAndDoubleEvenNumbers(numbers))
        {
            Console.WriteLine(number);
        }
    }
}
```

### Explanation

1. **Filter and Double Even Numbers:**
   ```csharp
   public static IEnumerable<int> FilterAndDoubleEvenNumbers(IEnumerable<int> numbers)
   ```
   - This named iterator method filters even numbers from the input and doubles them.

2. **Parameter Validation:**
   ```csharp
   if (numbers == null)
   {
       throw new ArgumentNullException(nameof(numbers));
   }
   ```
   - Checks if the input collection is null and throws an exception if it is.

3. **Filtering and Transformation:**
   ```csharp
   foreach (int number in numbers)
   {
       if (number % 2 == 0)
       {
           yield return number * 2;
       }
   }
   ```
   - Iterates through the input collection.
   - Checks if each number is even.
   - Uses `yield return` to return the doubled value of each even number.

### Conclusion

Named iterators in C# provide a powerful way to create custom iteration logic that is both readable and maintainable. By using the `yield` keyword, you can efficiently generate sequences of values without the need for intermediate collections, making your code more concise and efficient.