---
tags:
 - csharp
 - advanced
---

## What Is an Indexer?

An indexer lets you define **what `[]` means on your own class**. Without it, `[]` only works on arrays and built-in collections — your custom class wouldn’t support it.

Indexers are defined with the `this` keyword and can be used to provide array-like syntax for accessing elements in a class or struct.

---

## Why Use Indexers?

An indexer is essentially **syntactic sugar** — you could always achieve the same thing with regular `Get`/`Set` methods. But `[]` feels more natural for classes that conceptually represent a container:

```csharp
// Without indexer — works but awkward
string p = roster.GetPlayer(0);
roster.SetPlayer(0, "Alice");

// With indexer — clean and natural
string p = roster[0];
roster[0] = "Alice";

// Some things just feel right with []
var cell = spreadsheet["A1"];
var pixel = image[x, y];
var config = settings["theme"];
```

This is also why `Dictionary<TKey, TValue>`, `List<T>`, and arrays all use indexers — they’re containers, and `[]` is the natural way to access items in a container.

> [!note]
> Indexers are about **readability**, not capability. No one is forced to use them — you can always use methods instead.

---

## The Power — Custom Logic and Validation

An indexer doesn’t have to be a simple pass-through. You can add **your own logic** inside:

```csharp
class SafeList<T>
{
    private List<T> _items = new List<T>();

    public T this[int index]
    {
        get
        {
            if (index < 0 || index >= _items.Count)
                return default;  // return default instead of throwing
            return _items[index];
        }
        set
        {
            if (index < 0 || index >= _items.Count)
                throw new ArgumentOutOfRangeException();
            _items[index] = value;
        }
    }
}
```

You can also index by **non-integer types** like strings:

```csharp
class Team
{
    private Dictionary<string, Player> _players = new();

    public Player this[string name]
    {
        get => _players[name];
        set => _players[name] = value;
    }
}

var team = new Team();
team["Alice"] = new Player();
Player p = team["Alice"];
```

---

## Example: Simple Indexer
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