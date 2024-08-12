In C#, `IEnumerable` and `IEnumerator` are interfaces that support simple iteration over a collection of a specified type. They are fundamental to the way collections are accessed and manipulated in .NET. Here’s a detailed explanation of each interface, their roles, and how they work together.

### `IEnumerable<T>` Interface

The `IEnumerable<T>` interface is the base interface for all non-generic collections that can be enumerated. It is defined in the `System.Collections.Generic` namespace and provides a single method:

```csharp
public interface IEnumerable<out T>
{
    IEnumerator<T> GetEnumerator();
}
```

- **`GetEnumerator` Method**: This method returns an enumerator that iterates through the collection.

### `IEnumerator<T>` Interface

The `IEnumerator<T>` interface supports a simple iteration over a collection of a specified type. It is defined in the `System.Collections.Generic` namespace and includes the following members:

```csharp
public interface IEnumerator<out T> : IDisposable, IEnumerator
{
    T Current { get; }
}
```

The `IEnumerator<T>` interface itself extends the non-generic `IEnumerator` interface, which includes:

```csharp
public interface IEnumerator
{
    bool MoveNext();
    object Current { get; }
    void Reset();
}
```

- **`MoveNext` Method**: Advances the enumerator to the next element of the collection.
- **`Current` Property**: Gets the element in the collection at the current position of the enumerator.
- **`Reset` Method**: Sets the enumerator to its initial position, which is before the first element in the collection. Note that `Reset` is not always implemented and might throw a `NotSupportedException`.
- **`Dispose` Method**: Performs tasks associated with freeing, releasing, or resetting unmanaged resources (part of `IDisposable`).

### Relationship Between `IEnumerable<T>` and `IEnumerator<T>`

The `IEnumerable<T>` interface allows a collection to be enumerated using the `GetEnumerator` method, which returns an `IEnumerator<T>`. The `IEnumerator<T>` interface provides the functionality to traverse the collection.

### Example: Custom Collection with `IEnumerable<T>` and `IEnumerator<T>`

Here’s an example of a custom collection that implements `IEnumerable<T>` and a corresponding enumerator that implements `IEnumerator<T>`:

```csharp
using System;
using System.Collections;
using System.Collections.Generic;

public class CustomCollection<T> : IEnumerable<T>
{
    private T[] _items;

    public CustomCollection(T[] items)
    {
        _items = items;
    }

    public IEnumerator<T> GetEnumerator()
    {
        return new CustomEnumerator(_items);
    }

    IEnumerator IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }

    private class CustomEnumerator : IEnumerator<T>
    {
        private T[] _items;
        private int _index = -1;

        public CustomEnumerator(T[] items)
        {
            _items = items;
        }

        public bool MoveNext()
        {
            _index++;
            return _index < _items.Length;
        }

        public void Reset()
        {
            _index = -1;
        }

        public T Current
        {
            get
            {
                if (_index < 0 || _index >= _items.Length)
                    throw new InvalidOperationException();
                return _items[_index];
            }
        }

        object IEnumerator.Current => Current;

        public void Dispose()
        {
            // Clean up resources if necessary
        }
    }
}

public class Program
{
    public static void Main()
    {
        var collection = new CustomCollection<int>(new[] { 1, 2, 3, 4, 5 });

        foreach (var item in collection)
        {
            Console.WriteLine(item);
        }
    }
}
```

### Key Points

- **Encapsulation of Iteration Logic**: `IEnumerable<T>` encapsulates the logic to provide an enumerator, while `IEnumerator<T>` encapsulates the logic to iterate through the collection.
- **Use in `foreach` Loop**: `foreach` loops rely on `IEnumerable<T>` to get an enumerator and then use `IEnumerator<T>` to iterate over the collection.
- **Standard Pattern**: This pattern provides a standard way to iterate over collections, making it easy to work with different types of collections in a uniform manner.

By adhering to this pattern, custom collections can integrate seamlessly with the .NET framework's features like LINQ, `foreach` loops, and other collection-based operations.In C#, `IEnumerable` and `IEnumerator` are interfaces that support simple iteration over a collection of a specified type. They are fundamental to the way collections are accessed and manipulated in .NET. Here’s a detailed explanation of each interface, their roles, and how they work together.

### `IEnumerable<T>` Interface

The `IEnumerable<T>` interface is the base interface for all non-generic collections that can be enumerated. It is defined in the `System.Collections.Generic` namespace and provides a single method:

```csharp
public interface IEnumerable<out T>
{
    IEnumerator<T> GetEnumerator();
}
```

- **`GetEnumerator` Method**: This method returns an enumerator that iterates through the collection.

### `IEnumerator<T>` Interface

The `IEnumerator<T>` interface supports a simple iteration over a collection of a specified type. It is defined in the `System.Collections.Generic` namespace and includes the following members:

```csharp
public interface IEnumerator<out T> : IDisposable, IEnumerator
{
    T Current { get; }
}
```

The `IEnumerator<T>` interface itself extends the non-generic `IEnumerator` interface, which includes:

```csharp
public interface IEnumerator
{
    bool MoveNext();
    object Current { get; }
    void Reset();
}
```

- **`MoveNext` Method**: Advances the enumerator to the next element of the collection.
- **`Current` Property**: Gets the element in the collection at the current position of the enumerator.
- **`Reset` Method**: Sets the enumerator to its initial position, which is before the first element in the collection. Note that `Reset` is not always implemented and might throw a `NotSupportedException`.
- **`Dispose` Method**: Performs tasks associated with freeing, releasing, or resetting unmanaged resources (part of `IDisposable`).

### Relationship Between `IEnumerable<T>` and `IEnumerator<T>`

The `IEnumerable<T>` interface allows a collection to be enumerated using the `GetEnumerator` method, which returns an `IEnumerator<T>`. The `IEnumerator<T>` interface provides the functionality to traverse the collection.

### Example: Custom Collection with `IEnumerable<T>` and `IEnumerator<T>`

Here’s an example of a custom collection that implements `IEnumerable<T>` and a corresponding enumerator that implements `IEnumerator<T>`:

```csharp
using System;
using System.Collections;
using System.Collections.Generic;

public class CustomCollection<T> : IEnumerable<T>
{
    private T[] _items;

    public CustomCollection(T[] items)
    {
        _items = items;
    }

    public IEnumerator<T> GetEnumerator()
    {
        return new CustomEnumerator(_items);
    }

    IEnumerator IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }

    private class CustomEnumerator : IEnumerator<T>
    {
        private T[] _items;
        private int _index = -1;

        public CustomEnumerator(T[] items)
        {
            _items = items;
        }

        public bool MoveNext()
        {
            _index++;
            return _index < _items.Length;
        }

        public void Reset()
        {
            _index = -1;
        }

        public T Current
        {
            get
            {
                if (_index < 0 || _index >= _items.Length)
                    throw new InvalidOperationException();
                return _items[_index];
            }
        }

        object IEnumerator.Current => Current;

        public void Dispose()
        {
            // Clean up resources if necessary
        }
    }
}

public class Program
{
    public static void Main()
    {
        var collection = new CustomCollection<int>(new[] { 1, 2, 3, 4, 5 });

        foreach (var item in collection)
        {
            Console.WriteLine(item);
        }
    }
}
```

### Key Points

- **Encapsulation of Iteration Logic**: `IEnumerable<T>` encapsulates the logic to provide an enumerator, while `IEnumerator<T>` encapsulates the logic to iterate through the collection.
- **Use in `foreach` Loop**: `foreach` loops rely on `IEnumerable<T>` to get an enumerator and then use `IEnumerator<T>` to iterate over the collection.
- **Standard Pattern**: This pattern provides a standard way to iterate over collections, making it easy to work with different types of collections in a uniform manner.

By adhering to this pattern, custom collections can integrate seamlessly with the .NET framework's features like LINQ, `foreach` loops, and other collection-based operations.