The `IComparable` interface in C# is used to define a standard way to compare objects. Implementing this interface allows you to specify how instances of a class are ordered and compared. This is particularly useful for sorting collections of objects.

### `IComparable` Interface

The `IComparable` interface defines a single method:

```csharp
public interface IComparable
{
    int CompareTo(object obj);
}
```

The `CompareTo` method compares the current instance with another object of the same type. It returns an integer that indicates the relative position of the objects being compared.

- **Less than zero:** The current instance precedes the object specified by `obj` in the sort order.
- **Zero:** The current instance occurs in the same position in the sort order as the object specified by `obj`.
- **Greater than zero:** The current instance follows the object specified by `obj` in the sort order.

### Example Implementation

Here's an example of how to implement `IComparable` in a class:

```csharp
using System;

public class Person : IComparable<Person>
{
    public string Name { get; set; }
    public int Age { get; set; }

    public int CompareTo(Person other)
    {
        if (other == null) return 1;

        // Compare by Age first
        int ageComparison = this.Age.CompareTo(other.Age);

        if (ageComparison != 0)
        {
            return ageComparison;
        }
        else
        {
            // If Age is the same, compare by Name
            return this.Name.CompareTo(other.Name);
        }
    }

    public override string ToString()
    {
        return $"{Name}, {Age}";
    }
}

public class Program
{
    public static void Main()
    {
        Person[] people = {
            new Person { Name = "Alice", Age = 30 },
            new Person { Name = "Bob", Age = 25 },
            new Person { Name = "Charlie", Age = 30 },
        };

        Array.Sort(people);

        foreach (var person in people)
        {
            Console.WriteLine(person);
        }
    }
}
```

### Explanation

1. **Implement `IComparable<T>`:** The `Person` class implements `IComparable<Person>`. This generic version of the interface is preferred as it avoids the need for casting.
2. **Define `CompareTo` Method:** The `CompareTo` method first compares the `Age` property. If the ages are equal, it then compares the `Name` property. This ensures that people are first sorted by age and then by name if ages are equal.
3. **Sorting:** The `Array.Sort(people)` method uses the `CompareTo` method to sort the array of `Person` objects.

### Non-generic `IComparable`

If you need to implement the non-generic `IComparable` interface, it looks like this:

```csharp
public class Person : IComparable
{
    public string Name { get; set; }
    public int Age { get; set; }

    public int CompareTo(object obj)
    {
        if (obj == null) return 1;

        Person otherPerson = obj as Person;
        if (otherPerson != null)
        {
            // Compare by Age first
            int ageComparison = this.Age.CompareTo(otherPerson.Age);

            if (ageComparison != 0)
            {
                return ageComparison;
            }
            else
            {
                // If Age is the same, compare by Name
                return this.Name.CompareTo(otherPerson.Name);
            }
        }
        else
        {
            throw new ArgumentException("Object is not a Person");
        }
    }

    public override string ToString()
    {
        return $"{Name}, {Age}";
    }
}
```

### Summary

- **Purpose:** Implement `IComparable` to define a natural order for instances of your class.
- **Method:** Implement the `CompareTo` method to return an integer indicating the relative order of objects.
- **Generic vs. Non-generic:** Prefer the generic version (`IComparable<T>`) to avoid casting and improve type safety.
- **Usage:** Use `Array.Sort`, `List<T>.Sort`, or other sorting mechanisms to sort objects based on your `CompareTo` implementation.

By implementing `IComparable`, you enable objects of your class to be compared and sorted in a consistent and intuitive manner.