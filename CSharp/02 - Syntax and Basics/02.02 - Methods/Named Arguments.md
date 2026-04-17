---
tags:
 - csharp
 - basics
 - methods
---

Named arguments allow you to specify arguments by **parameter name** rather than relying on position. The syntax is `parameterName: value`.

```csharp
void PrintInfo(string name, int age, string city)
{
    Console.WriteLine($"Name: {name}, Age: {age}, City: {city}");
}

// Using named arguments — order does not matter
PrintInfo(age: 30, name: "John", city: "New York");

// Mixing positional and named — named must come after positional
PrintInfo("Jane", city: "Los Angeles", age: 25);
```

## Best Use Case: With Optional Arguments

Named arguments are most useful when a method has [[Optional Arguments|optional parameters]]. Without them, you would have to supply every preceding optional argument just to reach the one you want to set:

```csharp
void CreateUser(string name, int age = 0, string city = "Unknown", bool isAdmin = false)
{ ... }

// Without named arguments — must pass age and city just to set isAdmin
CreateUser("Alice", 0, "Unknown", true);

// With named arguments — skip straight to the one you care about
CreateUser("Alice", isAdmin: true);
```

This avoids filling in defaults you don't care about and makes the call site self-documenting.

## Rules

- Named arguments can appear in **any order**
- When mixing with positional arguments, **positional must come first**
- Named arguments make the call site explicit — the reader knows which parameter each value maps to without checking the method signature