---
tags:
 - csharp
 - basics
 - types
---

# Nullable Types in C\#

- C# has two kinds of nullable types: **nullable value types** and **nullable reference types**. They solve different problems but share the `?` syntax.

|                      | Nullable Value Types                               | Nullable Reference Types                                    |
| -------------------- | -------------------------------------------------- | ----------------------------------------------------------- |
| Available since      | C# 2.0                                             | C# 8.0                                                      |
| What it does         | Lets value types (`int`, `bool`, etc.) hold `null` | Lets the compiler warn when reference types might be `null` |
| Underlying mechanism | `Nullable<T>` struct wrapper                       | Compiler annotations and warnings (no runtime change)       |
| Default enabled      | Yes, always available                              | No, must opt in                                             |

---

## Nullable Value Types

In C#, value types (`int`, `bool`, `DateTime`, etc.) cannot normally be `null` as that is used to establish an empty object reference. Nullable value types allow them to represent missing or undefined values — useful for databases, optional fields, or APIs.

### Declaring Nullable Value Types

To declare a nullable value type, use `Nullable<T>` or the shorthand `T?`.

```csharp
int? nullableInt = null;          // Using the shorthand notation
Nullable<int> anotherNullableInt = null; // Using the Nullable<T> struct
```

### Key Properties and Methods

1. **HasValue**: Indicates whether the nullable type has a value.
2. **Value**: Gets the value if `HasValue` is `true`; otherwise, it throws an `InvalidOperationException`.
3. **GetValueOrDefault()**: Gets the value if `HasValue` is `true`; otherwise, it returns the default value for the type.

### Example

```csharp
using System;

class Program
{
    static void Main()
    {
        int? nullableInt = null;

        if (nullableInt.HasValue)
        {
            Console.WriteLine("Value: " + nullableInt.Value);
        }
        else
        {
            Console.WriteLine("Value is null");
        }

        // Assign a value
        nullableInt = 10;
        Console.WriteLine("Value after assignment: " + nullableInt);

        // Using the null-coalescing operator
        int valueOrDefault = nullableInt ?? -1;
        Console.WriteLine("Value or default: " + valueOrDefault);

        // Using the null-conditional operator
        int? result = nullableInt?.CompareTo(5);
        Console.WriteLine("Comparison result: " + result);
    }
}
```

### Practical Example: Optional Fields

```csharp
using System;

struct Person
{
    public string Name;
    public int? Age; // Age is optional

    public Person(string name, int? age = null)
    {
        Name = name;
        Age = age;
    }
}

class Program
{
    static void Main()
    {
        Person personWithAge = new Person("Alice", 30);
        Person personWithoutAge = new Person("Bob");

        Console.WriteLine($"{personWithAge.Name}, Age: {DisplayAge(personWithAge.Age)}");
        Console.WriteLine($"{personWithoutAge.Name}, Age: {DisplayAge(personWithoutAge.Age)}");
    }

    static string DisplayAge(int? age)
    {
        return age.HasValue ? age.Value.ToString() : "Unknown";
    }
}
```

### Practical Example: Database Operations

When working with databases, nullable fields are common. Here's an example using Entity Framework:

```csharp
using System;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

public class Employee
{
    [Key]
    public int Id { get; set; }
    
    [Required]
    public string Name { get; set; }
    
    public DateTime? HireDate { get; set; } // Nullable field
}

class Program
{
    static void Main()
    {
        using (var context = new EmployeeContext())
        {
            var employee = new Employee
            {
                Name = "John Doe",
                HireDate = null // This is allowed because HireDate is nullable
            };

            context.Employees.Add(employee);
            context.SaveChanges();

            var retrievedEmployee = context.Employees.Find(employee.Id);
            Console.WriteLine($"Employee Hire Date: {retrievedEmployee.HireDate?.ToString() ?? "Not Hired"}");
        }
    }
}

public class EmployeeContext : DbContext
{
    public DbSet<Employee> Employees { get; set; }
}
```

---

## Nullable Reference Types (C# 8.0+)

Nullable reference types were introduced in C# 8.0 to enhance the language's ability to handle `null` references safely and explicitly. This feature allows developers to distinguish between reference types that can be `null` and those that cannot, reducing the likelihood of `NullReferenceException` at runtime.

```ad-important
Unlike nullable value types, nullable reference types are a **compile-time feature only**. They produce warnings, not errors (by default), and there is no runtime wrapper — a `string?` is still just a `string` at runtime.
```

### Enabling Nullable Reference Types

Nullable reference types are not enabled by default. You can enable them per-file or project-wide.

#### Per-file

```csharp
#nullable enable

class Program
{
    static void Main()
    {
        string? nullableString = null; // Nullable reference type
        string nonNullableString = "Hello"; // Non-nullable reference type

        if (nullableString != null)
        {
            Console.WriteLine(nullableString.Length); // Safe to use
        }

        Console.WriteLine(nonNullableString.Length); // Always safe
    }
}
```

#### Project-wide

In your `.csproj` file:

```xml
<PropertyGroup>
    <Nullable>enable</Nullable>
</PropertyGroup>
```

### Nullable Annotations

- `T?`: Indicates that a variable of reference type `T` can be `null`.
- `T`: Indicates that a variable of reference type `T` cannot be `null`.

### Example

```csharp
#nullable enable

class Person
{
    public string Name { get; set; } // Non-nullable reference type
    public string? MiddleName { get; set; } // Nullable reference type

    public Person(string name)
    {
        Name = name;
    }

    public void PrintMiddleName()
    {
        if (MiddleName != null)
        {
            Console.WriteLine(MiddleName); // Safe to use
        }
        else
        {
            Console.WriteLine("No middle name provided.");
        }
    }
}

class Program
{
    static void Main()
    {
        Person person = new Person("John");
        person.PrintMiddleName(); // Prints "No middle name provided."

        person.MiddleName = "William";
        person.PrintMiddleName(); // Prints "William"
    }
}
```

### Nullable Contexts

Nullable contexts can be enabled or disabled at various scopes using the `#nullable` directive:

- `#nullable enable`: Enables nullable annotations and warnings.
- `#nullable disable`: Disables nullable annotations and warnings.
- `#nullable restore`: Restores the nullable context to the project-wide setting.

```csharp
#nullable enable

class Example
{
    public void Method()
    {
        string? nullableString = null; // Nullable context enabled
        string nonNullableString = "Hello";

        #nullable disable
        string anotherString = null; // Nullable context disabled (no warning)

        #nullable restore
        string restoredContextString = null; // Nullable context restored (warning if null)
    }
}
```

### Suppressing Nullable Warnings

If you need to suppress a nullable warning, use the null-forgiving operator (`!`). This tells the compiler that you are certain the value is not `null`, even if it cannot verify that.

```csharp
#nullable enable

class Example
{
    public string GetValue(string? input)
    {
        return input!; // Null-forgiving operator used, no warning
    }
}
```

---

## Best Practices

1. **Use Nullable Annotations**: Annotate your APIs and internal methods with nullable types to indicate which parameters and return values can be `null`.
2. **Check for Null**: Always check for `null` when dealing with nullable types before accessing their members.
3. **Avoid Suppressing Warnings**: Use the null-forgiving operator (`!`) sparingly and only when you are absolutely sure the value will not be `null`.
4. **Enable Project-wide**: Prefer enabling nullable reference types at the project level in `.csproj` rather than per-file, for consistency.
