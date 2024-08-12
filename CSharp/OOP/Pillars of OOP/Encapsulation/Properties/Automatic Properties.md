Automatic properties in C# provide a simplified syntax for property declaration. With automatic properties, you don't need to define a backing field explicitly. The compiler generates a private, anonymous backing field that can be accessed through the property's get and set accessors.

### Basic Syntax
Here's how you define automatic properties in C#:

```csharp
public class Person
{
    // Automatic property with get and set accessors
    public string Name { get; set; }

    // Automatic property with a private setter
    public int Age { get; private set; }

    // Automatic property with a default value
    public string Country { get; set; } = "Unknown";
}
```

### Explanation
1. **`public string Name { get; set; }`**: This defines a `Name` property with both get and set accessors. The compiler generates the backing field automatically.
2. **`public int Age { get; private set; }`**: This defines an `Age` property where the set accessor is private. This means the property can only be set within the class but can be read from outside.
3. **`public string Country { get; set; } = "Unknown";`**: This defines a `Country` property with a default value. When a new `Person` object is created, the `Country` property is initialized to `"Unknown"` unless explicitly set to another value.

### Example Usage

```csharp
public class Program
{
    public static void Main()
    {
        Person person = new Person
        {
            Name = "John Doe",
            Age = 30, // Cannot set directly outside the class if Age has a private setter
            Country = "USA"
        };

        // Accessing the properties
        Console.WriteLine($"Name: {person.Name}");
        Console.WriteLine($"Age: {person.Age}");
        Console.WriteLine($"Country: {person.Country}");
    }
}
```

### Benefits
- **Simplicity**: Simplifies the code by reducing boilerplate code required for properties.
- **Readability**: Makes the code easier to read and maintain.
- **Encapsulation**: Provides encapsulation with minimal syntax, making it clear where properties are set and retrieved.

### Full Example with Default Values and Encapsulation

Here's a more comprehensive example that demonstrates automatic properties with default values and encapsulation:

```csharp
public class Employee
{
    // Automatic property with default value
    public string Position { get; set; } = "Employee";

    // Automatic property with private setter
    public decimal Salary { get; private set; }

    // Constructor to initialize properties
    public Employee(string name, decimal initialSalary)
    {
        Name = name;
        Salary = initialSalary;
    }

    // Automatic property with only get accessor
    public string Name { get; }

    // Method to give a raise
    public void GiveRaise(decimal amount)
    {
        if (amount > 0)
        {
            Salary += amount;
        }
    }
}

public class Program
{
    public static void Main()
    {
        Employee employee = new Employee("Jane Doe", 50000m);

        // Accessing properties
        Console.WriteLine($"Name: {employee.Name}");
        Console.WriteLine($"Position: {employee.Position}");
        Console.WriteLine($"Salary: {employee.Salary}");

        // Giving a raise
        employee.GiveRaise(5000m);
        Console.WriteLine($"New Salary: {employee.Salary}");
    }
}
```

### Summary
Automatic properties in C# streamline the syntax for defining properties by allowing the compiler to generate the backing fields. They improve code clarity and maintainability, making it easier to work with encapsulated data.