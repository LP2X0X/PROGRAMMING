In C#, there are several ways to format strings. These include using string concatenation, `String.Format`, interpolated strings, and various formatting methods provided by the `String` class. Here are the most commonly used methods:

### 1. String Concatenation

String concatenation involves using the `+` operator to join strings together.

```csharp
string firstName = "John";
string lastName = "Doe";
string fullName = "Hello, " + firstName + " " + lastName + "!";
Console.WriteLine(fullName);  // Output: Hello, John Doe!
```

### 2. String.Format

The `String.Format` method allows for more complex string formatting and is particularly useful for embedding values into strings.

```csharp
string firstName = "John";
string lastName = "Doe";
string fullName = String.Format("Hello, {0} {1}!", firstName, lastName);
Console.WriteLine(fullName);  // Output: Hello, John Doe!
```

### 3. String Interpolation

String interpolation is a more readable and concise way to format strings, introduced in C# 6.0. It uses the `$` symbol before the string and curly braces `{}` to embed expressions.

```csharp
string firstName = "John";
string lastName = "Doe";
string fullName = $"Hello, {firstName} {lastName}!";
Console.WriteLine(fullName);  // Output: Hello, John Doe!
```

### 4. Composite Formatting

Composite formatting uses placeholders in a format string, which are replaced by the string representations of corresponding objects.

```csharp
string firstName = "John";
string lastName = "Doe";
string fullName = string.Format("Hello, {0} {1}!", firstName, lastName);
Console.WriteLine(fullName);  // Output: Hello, John Doe!
```

### 5. String.Join

`String.Join` can be used to concatenate elements of a string array with a specified separator.

```csharp
string[] names = { "John", "Doe" };
string fullName = string.Join(" ", names);
Console.WriteLine(fullName);  // Output: John Doe
```

### 6. StringBuilder

`StringBuilder` is used for creating and manipulating large strings efficiently.

```csharp
using System.Text;

StringBuilder sb = new StringBuilder();
sb.Append("Hello, ");
sb.Append("John ");
sb.Append("Doe!");
string fullName = sb.ToString();
Console.WriteLine(fullName);  // Output: Hello, John Doe!
```

### 7. String.Concat

`String.Concat` can concatenate multiple strings or string representations of objects.

```csharp
string firstName = "John";
string lastName = "Doe";
string fullName = string.Concat("Hello, ", firstName, " ", lastName, "!");
Console.WriteLine(fullName);  // Output: Hello, John Doe!
```

### 8. String.PadLeft and String.PadRight

These methods add padding to a string to ensure it reaches a specified length.

```csharp
string value = "123";
string paddedValue = value.PadLeft(5, '0');
Console.WriteLine(paddedValue);  // Output: 00123
```

### 9. String.Format with CultureInfo

You can format numbers, dates, and other values using specific cultural settings with `String.Format`.

```csharp
using System;
using System.Globalization;

double value = 1234.56;
string formattedValue = String.Format(new CultureInfo("en-US"), "{0:C}", value);
Console.WriteLine(formattedValue);  // Output: $1,234.56
```

### Summary

These methods provide a range of options to format strings in C#, from simple concatenation to more complex formatting scenarios. String interpolation (`$"..."`) is often preferred for its readability and simplicity, while `StringBuilder` is useful for scenarios involving extensive or complex string manipulations.