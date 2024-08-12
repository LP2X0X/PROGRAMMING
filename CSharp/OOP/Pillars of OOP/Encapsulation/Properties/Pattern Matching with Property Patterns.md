Pattern Matching with Property Patterns is a feature in C# that allows for more concise and readable code when working with objects. This feature extends pattern matching by enabling you to match objects based on their properties, rather than just their type or value.

### Basic Syntax

Here is the basic syntax for property patterns:

```csharp
if (obj is { Property1: value1, Property2: value2 })
{
    // code to execute if the pattern matches
}
```

In this pattern, `obj` is an object being matched against the specified properties and their values.

### Examples

#### Example 1: Simple Property Pattern

Let's consider a simple example where we have a `Person` class:

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}
```

We can use a property pattern to match a `Person` object with a specific name and age:

```csharp
Person person = new Person { Name = "John", Age = 30 };

if (person is { Name: "John", Age: 30 })
{
    Console.WriteLine("Matched John who is 30 years old.");
}
```

#### Example 2: Nested Property Patterns

Property patterns can be nested, allowing you to match properties of properties:

```csharp
public class Address
{
    public string City { get; set; }
    public string Street { get; set; }
}

public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    public Address Address { get; set; }
}

Person person = new Person 
{ 
    Name = "John", 
    Age = 30, 
    Address = new Address { City = "New York", Street = "5th Avenue" } 
};

if (person is { Address: { City: "New York" } })
{
    Console.WriteLine("John lives in New York.");
}
```

In this example, the pattern matches if the `Person` object has an `Address` property where the `City` is "New York".

### Using Variables in Property Patterns

You can also use variables within property patterns to capture property values:

```csharp
if (person is { Name: var name, Age: var age })
{
    Console.WriteLine($"Matched a person named {name} who is {age} years old.");
}
```

### Combining Property Patterns with Logical Patterns

Property patterns can be combined with logical patterns such as `and`, `or`, and `not`:

```csharp
if (person is { Name: "John", Age: >= 30 })
{
    Console.WriteLine("Matched John who is 30 years old or older.");
}
```

### Example: Switch Expression with Property Patterns

Property patterns can also be used in switch expressions to handle different cases based on properties:

```csharp
string GetPersonDescription(Person person) => person switch
{
    { Name: "John", Age: 30 } => "John is 30 years old.",
    { Age: > 30 } => "This person is older than 30.",
    { Age: < 30 } => "This person is younger than 30.",
    _ => "Unknown person."
};
```

### Property patterns can be nested to navigate down the property chain
In C#, property patterns can be nested to navigate down the property chain by specifying patterns for properties of properties. This allows you to match and deconstruct complex objects in a concise and readable manner. Here's a detailed explanation of how this works with an example:

#### Basic Concept
A property pattern is used to match properties of an object. When these properties are themselves objects, you can further match their properties using nested property patterns.

#### Syntax
The syntax for nested property patterns is an extension of the basic property pattern syntax. You simply continue to write property patterns for nested properties within the outer property pattern.

#### Example
Let's consider a scenario where you have the following classes:

```csharp
public class Address
{
    public string City { get; set; }
    public string Country { get; set; }
}

public class Person
{
    public string Name { get; set; }
    public Address Address { get; set; }
}
```

You want to match a `Person` object with a specific `City` in their `Address`. Here is how you can use nested property patterns to achieve this:

```csharp
public void CheckPerson(Person person)
{
    if (person is { Address: { City: "New York" } })
    {
        Console.WriteLine("The person lives in New York.");
    }
    else
    {
        Console.WriteLine("The person does not live in New York.");
    }
}
```

#### Explanation
1. **Outer Pattern (`person is { Address: ... }`)**: This checks if the `person` object has a non-null `Address` property.
2. **Nested Pattern (`Address: { City: "New York" }`)**: This further checks if the `Address` property has a `City` property with the value `"New York"`.

#### More Complex Example
You can nest multiple levels and match multiple properties:

```csharp
public void CheckPersonDetailed(Person person)
{
    if (person is { Name: "John Doe", Address: { City: "New York", Country: "USA" } })
    {
        Console.WriteLine("John Doe lives in New York, USA.");
    }
    else
    {
        Console.WriteLine("It's not John Doe in New York, USA.");
    }
}
```

#### Explanation
1. **`Name: "John Doe"`**: Matches if the `Name` property of `person` is `"John Doe"`.
2. **`Address: { City: "New York", Country: "USA" }`**: Matches if the `Address` property of `person` has a `City` property with the value `"New York"` and a `Country` property with the value `"USA"`.

#### Combining with Other Patterns
You can combine nested property patterns with other pattern types like type patterns or positional patterns for more complex matching scenarios.

```csharp
public void CheckPersonComplex(Person person)
{
    if (person is { Name: string name, Address: { City: "New York", Country: "USA" } })
    {
        Console.WriteLine($"Person {name} lives in New York, USA.");
    }
}
```

#### Extended Property Patterns
New in C# 10, extended property patterns can be used instead of nesting downstream properties. This update cleans up the previous example, as shown here:  
```csharp
public void GiveBonus(float amount)  
{  
	Pay = this switch  
	{  
		{ Age: >= 18, PayType: EmployeePayTypeEnum.Commission, HireDate.Year: > 2020 }  
			=> Pay += .10F * amount,  
		{ Age: >= 18, PayType: EmployeePayTypeEnum.Hourly, HireDate.Year: > 2020 }  
			=> Pay += 40F * amount / 2080F,  
		{ Age: >= 18, PayType: EmployeePayTypeEnum.Salaried, HireDate.Year: > 2020 }  
			=> Pay += amount,  
		_ => Pay += 0  
	};  
}
```

#### Summary
Nested property patterns in C# allow you to concisely and effectively match complex object structures by specifying patterns for properties of properties. This feature enhances the readability and expressiveness of pattern matching in C#.

### Benefits of Property Patterns

1. **Readability**: Property patterns make the code more readable and expressive.
2. **Conciseness**: They allow for concise matching and extraction of property values.
3. **Flexibility**: They provide flexibility to match objects based on nested properties and combinations of patterns.

### Conclusion

Property patterns in C# are a powerful feature that enhance pattern matching capabilities, making it easier to work with objects based on their properties. By leveraging property patterns, you can write more readable, maintainable, and expressive code.