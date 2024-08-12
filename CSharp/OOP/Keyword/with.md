In C#, the `with` keyword is used with records to perform non-destructive mutation. This means you can create a new record instance that is a copy of an existing one, but with some properties modified. This allows you to work with immutable data structures while still being able to make changes when needed.

### Syntax

The `with` expression creates a new record by copying the existing record and modifying specified properties. Here's the basic syntax:

```csharp
var newRecord = existingRecord with { PropertyName = newValue, AnotherProperty = anotherValue };
```

### Example

Let's consider a simple record type:

```csharp
public record Person(string FirstName, string LastName, int Age);
```

Now, we can create an instance of `Person` and use the `with` expression to create a new instance with some properties changed:

```csharp
public class Program
{
    public static void Main()
    {
        var person1 = new Person("John", "Doe", 30);

        // Using the 'with' keyword to create a new record with a modified property
        var person2 = person1 with { Age = 31 };

        // Output the original and the modified record
        Console.WriteLine(person1); // Outputs: Person { FirstName = John, LastName = Doe, Age = 30 }
        Console.WriteLine(person2); // Outputs: Person { FirstName = John, LastName = Doe, Age = 31 }

        // Verify that person1 and person2 are different instances
        Console.WriteLine(object.ReferenceEquals(person1, person2)); // Outputs: False
    }
}
```

### Explanation

1. **Original Record (`person1`)**: The original record instance is created with the values "John", "Doe", and 30 for `FirstName`, `LastName`, and `Age` respectively.
2. **Modified Record (`person2`)**: The `with` expression creates a new `Person` record, copying all properties from `person1` but changing the `Age` property to 31.
3. **Immutability**: The original record (`person1`) remains unchanged, demonstrating the immutability of records.
4. **Reference Equality**: The `object.ReferenceEquals` method confirms that `person1` and `person2` are different instances, even though they share most of their data.

### More Complex Example

For a more complex scenario, consider a record with nested records:

```csharp
public record Address(string City, string Country);

public record Person(string FirstName, string LastName, int Age, Address Address);
```

Now, let's use the `with` expression to create a modified copy of a `Person` record, including changes to a nested `Address` record:

```csharp
public class Program
{
    public static void Main()
    {
        var address = new Address("New York", "USA");
        var person1 = new Person("John", "Doe", 30, address);

        // Using the 'with' keyword to create a new record with a modified nested property
        var person2 = person1 with { Address = person1.Address with { City = "Los Angeles" } };

        // Output the original and the modified records
        Console.WriteLine(person1); // Outputs: Person { FirstName = John, LastName = Doe, Age = 30, Address = Address { City = New York, Country = USA } }
        Console.WriteLine(person2); // Outputs: Person { FirstName = John, LastName = Doe, Age = 30, Address = Address { City = Los Angeles, Country = USA } }

        // Verify that person1 and person2 are different instances
        Console.WriteLine(object.ReferenceEquals(person1, person2)); // Outputs: False

        // Verify that the nested address objects are different instances
        Console.WriteLine(object.ReferenceEquals(person1.Address, person2.Address)); // Outputs: False
    }
}
```

### Summary

- **Non-Destructive Mutation**: The `with` expression allows you to create modified copies of records without changing the original instance.
- **Immutability**: Records remain immutable, promoting safer and more predictable code.
- **Nested Records**: The `with` expression works recursively, allowing modifications of nested records as well.

The `with` keyword in C# records provides a powerful way to work with immutable data structures, enabling you to make changes by creating new instances with the desired modifications.