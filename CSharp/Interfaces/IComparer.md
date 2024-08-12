The `IComparer` interface in C# is used to provide custom comparison logic for sorting objects. Unlike `IComparable`, which defines a default comparison method within a class, `IComparer` allows you to define multiple comparison strategies outside of the class. This is useful when you want to sort objects in different ways without modifying the objects themselves.

### `IComparer` Interface

The `IComparer` interface defines a single method:

```csharp
public interface IComparer
{
    int Compare(object x, object y);
}
```

The `Compare` method compares two objects and returns an integer indicating their relative order:

- **Less than zero:** `x` is less than `y`.
- **Zero:** `x` is equal to `y`.
- **Greater than zero:** `x` is greater than `y`.

### Generic Version: `IComparer<T>`

The generic version of `IComparer` is `IComparer<T>`, which provides type safety and avoids the need for casting.

### Example Implementation

Here's an example of how to implement `IComparer<T>`:

#### 1. Define a Class to Compare

First, define the class you want to compare:

```csharp
using System;

public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }

    public override string ToString()
    {
        return $"{Name}, {Age}";
    }
}
```

#### 2. Implement Custom Comparer Classes

Next, implement custom comparers for different sorting criteria. For instance, you can create comparers for sorting by age and by name.

##### Comparer by Age

```csharp
using System.Collections.Generic;

public class AgeComparer : IComparer<Person>
{
    public int Compare(Person x, Person y)
    {
        if (x == null || y == null)
        {
            throw new ArgumentException("Arguments cannot be null");
        }

        return x.Age.CompareTo(y.Age);
    }
}
```

##### Comparer by Name

```csharp
using System.Collections.Generic;

public class NameComparer : IComparer<Person>
{
    public int Compare(Person x, Person y)
    {
        if (x == null || y == null)
        {
            throw new ArgumentException("Arguments cannot be null");
        }

        return string.Compare(x.Name, y.Name);
    }
}
```

#### 3. Using the Comparers

Finally, use these comparers to sort a list of `Person` objects:

```csharp
using System;
using System.Collections.Generic;

public class Program
{
    public static void Main()
    {
        List<Person> people = new List<Person>
        {
            new Person { Name = "Alice", Age = 30 },
            new Person { Name = "Bob", Age = 25 },
            new Person { Name = "Charlie", Age = 30 }
        };

        Console.WriteLine("Original List:");
        foreach (var person in people)
        {
            Console.WriteLine(person);
        }

        people.Sort(new AgeComparer());
        Console.WriteLine("\nSorted by Age:");
        foreach (var person in people)
        {
            Console.WriteLine(person);
        }

        people.Sort(new NameComparer());
        Console.WriteLine("\nSorted by Name:");
        foreach (var person in people)
        {
            Console.WriteLine(person);
        }
    }
}
```

### Explanation

1. **Person Class:** Defines the `Person` class with `Name` and `Age` properties.
2. **AgeComparer Class:** Implements `IComparer<Person>` to compare `Person` objects by age.
3. **NameComparer Class:** Implements `IComparer<Person>` to compare `Person` objects by name.
4. **Sorting:** Uses the `Sort` method of the `List<T>` class with the custom comparers to sort the list of `Person` objects by age and by name.

### Summary

- **Purpose:** Use `IComparer` to define custom comparison logic outside of the objects being compared.
- **Method:** Implement the `Compare` method to return an integer indicating the relative order of two objects.
- **Generic Version:** Prefer `IComparer<T>` to avoid casting and improve type safety.
- **Usage:** Create custom comparer classes for different sorting criteria and use them with sorting methods like `List<T>.Sort`.

Using `IComparer`, you can easily switch between different sorting strategies without modifying the classes being sorted, providing flexibility and separation of concerns in your code.