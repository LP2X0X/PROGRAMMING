- The first way to use the TPL is to perform data parallelism. Simply put, this term refers to the task of iterating over an array or collection in a parallel manner using the Parallel.For() or Parallel.ForEach() method.
```ad-note
When you are building a multithreaded graphical user interface (gui) application, secondary threads can never directly access user interface controls. the reason is that controls (buttons, text boxes, labels, progress bars, etc.) have thread affinity with the thread that created them.
```

### Key Components of Data Parallelism in TPL

1. **Parallel Class**: Provides methods for parallel operations on collections, such as `For`, `ForEach`, and `Invoke`.
2. **PLINQ (Parallel LINQ)**: Extends LINQ with parallel operations for querying and manipulating data collections.

### Using the `Parallel` Class

The `Parallel` class in TPL provides straightforward methods to parallelize loops and execute actions concurrently.

#### Example: Parallel.For

The `Parallel.For` method is used to parallelize a `for` loop.

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        int[] numbers = new int[10000];
        Parallel.For(0, numbers.Length, i =>
        {
            numbers[i] = i * i;
        });

        Console.WriteLine("All numbers processed.");
    }
}
```

In this example, the `Parallel.For` method executes the loop body in parallel, distributing iterations across multiple threads.

#### Example: Parallel.ForEach

The `Parallel.ForEach` method is used to parallelize operations on a collection.

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        var numbers = new List<int>(10000);
        for (int i = 0; i < 10000; i++)
        {
            numbers.Add(i);
        }

        Parallel.ForEach(numbers, number =>
        {
            Console.WriteLine($"Processing number {number}");
        });

        Console.WriteLine("All numbers processed.");
    }
}
```

In this example, `Parallel.ForEach` processes each item in the `numbers` list in parallel.

### Using PLINQ

PLINQ (Parallel LINQ) extends the capabilities of LINQ by providing parallel query execution. It allows you to process large data collections more efficiently by leveraging multiple cores.

#### Example: Basic PLINQ Query

Here's an example of using PLINQ to process a collection in parallel.

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        var numbers = Enumerable.Range(0, 10000);

        var squaredNumbers = numbers.AsParallel()
                                    .Select(number => number * number)
                                    .ToArray();

        Console.WriteLine("All numbers processed.");
    }
}
```

In this example, `AsParallel` converts the `numbers` collection into a parallel query, allowing the `Select` method to execute in parallel.

#### Example: Controlling Parallelism in PLINQ

You can control the degree of parallelism and other settings in PLINQ using the `WithDegreeOfParallelism` method.

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        var numbers = Enumerable.Range(0, 10000);

        var squaredNumbers = numbers.AsParallel()
                                    .WithDegreeOfParallelism(4) // Use 4 parallel tasks
                                    .Select(number => number * number)
                                    .ToArray();

        Console.WriteLine("All numbers processed.");
    }
}
```

### Considerations for Data Parallelism

1. **Thread Safety**: Ensure that operations within parallel loops are thread-safe. Avoid shared state or use synchronization mechanisms.
2. **Partitioning**: TPL and PLINQ automatically partition the data for parallel processing. However, understanding how data is partitioned can help in optimizing performance.
3. **Overhead**: Parallel execution involves overhead. For small datasets or lightweight operations, the overhead might outweigh the benefits.
4. **Exception Handling**: Handle exceptions in parallel operations using aggregate exceptions (`AggregateException`).

#### Example: Handling Exceptions in Parallel.ForEach

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        var numbers = new List<int> { 1, 2, 3, 4, 5 };

        try
        {
            Parallel.ForEach(numbers, number =>
            {
                if (number == 3)
                {
                    throw new InvalidOperationException("An error occurred.");
                }

                Console.WriteLine($"Processing number {number}");
            });
        }
        catch (AggregateException ex)
        {
            foreach (var innerEx in ex.InnerExceptions)
            {
                Console.WriteLine(innerEx.Message);
            }
        }

        Console.WriteLine("All numbers processed.");
    }
}
```

### Summary

Data parallelism in TPL using the `Parallel` class and PLINQ allows for efficient parallel processing of collections. By leveraging multiple cores, you can significantly improve the performance of compute-intensive operations. Ensure thread safety, manage overhead, and handle exceptions appropriately to fully benefit from parallel execution.