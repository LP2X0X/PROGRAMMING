In C#, an indexer allows an object to be indexed in the same way as an array. Indexers are defined with the `this` keyword and can be used to provide array-like syntax for accessing elements in a class or struct. This is particularly useful for creating classes that represent collections or containers of data.

Here’s a simple example of a class with an indexer:

### Example: Simple Indexer
```csharp
using System;

class SampleCollection<T>
{
    private T[] array = new T[100]; // Internal array to hold data

    // Indexer declaration
    public T this[int index]
    {
        get
        {
            // Perform bounds check
            if (index < 0 || index >= array.Length)
            {
                throw new IndexOutOfRangeException("Index out of range");
            }
            return array[index]; // Get the value at the specified index
        }
        set
        {
            // Perform bounds check
            if (index < 0 || index >= array.Length)
            {
                throw new IndexOutOfRangeException("Index out of range");
            }
            array[index] = value; // Set the value at the specified index
        }
    }
}

class Program
{
    static void Main()
    {
        var collection = new SampleCollection<string>();

        // Use the indexer to set values
        collection[0] = "Hello";
        collection[1] = "World";

        // Use the indexer to get values
        Console.WriteLine(collection[0]); // Output: Hello
        Console.WriteLine(collection[1]); // Output: World
    }
}
```

### Explanation:
1. **Class Declaration**: `SampleCollection<T>` is a generic class that can hold elements of any type `T`.
2. **Private Array**: A private array `array` of type `T` is used to store the elements.
3. **Indexer Declaration**: The indexer is declared using the `this` keyword followed by an index parameter in square brackets.
4. **Getter**: The `get` accessor returns the value at the specified index after performing a bounds check.
5. **Setter**: The `set` accessor sets the value at the specified index after performing a bounds check.
6. **Usage**: In the `Main` method, the indexer is used to set and get values from the `SampleCollection<string>` instance.

### Advanced Example: Multi-Dimensional Indexer
You can also define indexers with multiple parameters for multi-dimensional access:

```csharp
using System;

class Matrix
{
    private int[,] matrix = new int[10, 10]; // 2D array to hold data

    // Indexer for 2D array
    public int this[int row, int column]
    {
        get
        {
            return matrix[row, column]; // Get the value at the specified row and column
        }
        set
        {
            matrix[row, column] = value; // Set the value at the specified row and column
        }
    }
}

class Program
{
    static void Main()
    {
        var matrix = new Matrix();

        // Use the indexer to set values
        matrix[0, 0] = 1;
        matrix[0, 1] = 2;
        matrix[1, 0] = 3;
        matrix[1, 1] = 4;

        // Use the indexer to get values
        Console.WriteLine(matrix[0, 0]); // Output: 1
        Console.WriteLine(matrix[0, 1]); // Output: 2
        Console.WriteLine(matrix[1, 0]); // Output: 3
        Console.WriteLine(matrix[1, 1]); // Output: 4
    }
}
```

### Explanation:
1. **Class Declaration**: `Matrix` is a class that encapsulates a 2D array of integers.
2. **Private 2D Array**: A private 2D array `matrix` is used to store the elements.
3. **Indexer Declaration**: The indexer is declared with two parameters for row and column.
4. **Getter and Setter**: The `get` and `set` accessors access elements in the 2D array using the row and column indices.

Indexers provide a way to create classes that are intuitive to use and integrate seamlessly with array-like access patterns.