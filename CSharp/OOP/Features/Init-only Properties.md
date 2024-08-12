https://stackoverflow.com/questions/62372422/what-is-difference-between-init-only-and-readonly-in-c-sharp-9

---
In C#, both `init-only` properties and `readonly` fields are used to create immutable objects or ensure that certain values cannot be changed after initialization. However, they serve different purposes and have different use cases. Here's a detailed comparison:

### `readonly` Fields
- **Definition**: A `readonly` field is a field that can only be assigned a value during its declaration or in the constructor of the class.
- **Usage**: Typically used to initialize values that should not change after the object is constructed.
- **Assignment**: Can only be assigned in the field declaration or within a constructor.
- **Example**:
  ```csharp
  public class Example
  {
      public readonly int ReadOnlyValue;

      public Example(int value)
      {
          ReadOnlyValue = value;
      }
  }
  ```

### `init-only` Properties
- **Definition**: An `init-only` property is a property that can be set during object initialization but cannot be modified afterwards. Introduced in C# 9.0.
- **Usage**: Used to provide immutability for properties while still allowing the use of object initializers.
- **Assignment**: Can be assigned only during object initialization using an object initializer.
- **Example**:
  ```csharp
  public class Example
  {
      public int InitOnlyValue { get; init; }
  }
  ```

### Key Differences

#### 1. **Assignment Timing**
- **`readonly` Fields**: Can be assigned during declaration or in the constructor.
- **`init-only` Properties**: Can be assigned during object initialization using an object initializer.

#### 2. **Flexibility**
- **`readonly` Fields**: Provide more flexibility in terms of where they can be set (declaration or constructor).
- **`init-only` Properties**: Are specifically designed for use with object initializers and can make code more readable and concise when creating immutable objects.

#### 3. **Usage with Object Initializers**
- **`readonly` Fields**: Cannot be used with object initializers directly.
- **`init-only` Properties**: Are specifically designed to work with object initializers, allowing for a more fluent and expressive syntax.

#### 4. **Encapsulation**
- **`readonly` Fields**: Expose the field directly, which may not be ideal for encapsulation.
- **`init-only` Properties**: Use properties, which are better suited for encapsulation and can include additional logic like validation or lazy loading if needed.

### Example Comparison

Here’s an example that compares `readonly` fields and `init-only` properties:

#### Using `readonly` Fields:
```csharp
public class Person
{
    public readonly string FirstName;
    public readonly string LastName;

    public Person(string firstName, string lastName)
    {
        FirstName = firstName;
        LastName = lastName;
    }
}

var person = new Person("John", "Doe");
Console.WriteLine($"{person.FirstName} {person.LastName}");
```

#### Using `init-only` Properties:
```csharp
public class Person
{
    public string FirstName { get; init; }
    public string LastName { get; init; }
}

var person = new Person { FirstName = "John", LastName = "Doe" };
Console.WriteLine($"{person.FirstName} {person.LastName}");
```

### Summary
- **`readonly` Fields**: Ideal for values that should not change after object construction and need to be set within constructors.
- **`init-only` Properties**: Ideal for creating immutable objects with a clean and expressive syntax using object initializers.

Both features help enforce immutability but are used in different contexts and offer different advantages in terms of syntax and flexibility.