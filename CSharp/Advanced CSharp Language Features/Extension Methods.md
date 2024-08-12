Extension methods in C# allow you to add new methods to existing types (classes, structs, interfaces) without modifying the original type or creating a new derived type. They are defined as static methods in a static class, and they are used as if they were instance methods of the extended type. Here's a detailed explanation and example of how extension methods work in C#:

### Syntax

To define an extension method:

```csharp
public static class ExtensionClass
{
    public static ReturnType MethodName(this ExtendedType extendedParam, otherParams)
    {
        // Method implementation
    }
}
```

Where:
- `ExtensionClass` is a static class containing the extension method.
- `ReturnType` is the return type of the method.
- `MethodName` is the name of the extension method.
- `this ExtendedType extendedParam` is the first parameter, prefixed with `this`, specifying the type being extended.
- `otherParams` are additional parameters for the method.

### Example

Let's create an extension method for the `string` type that converts a string to title case (capitalizes the first letter of each word):

```csharp
public static class StringExtensions
{
    public static string ToTitleCase(this string str)
    {
        if (string.IsNullOrEmpty(str))
            return str;

        string[] words = str.Split(' ');
        for (int i = 0; i < words.Length; i++)
        {
            if (!string.IsNullOrEmpty(words[i]))
            {
                char[] letters = words[i].ToCharArray();
                if (letters.Length > 0)
                {
                    letters[0] = char.ToUpper(letters[0]);
                }
                words[i] = new string(letters);
            }
        }
        return string.Join(" ", words);
    }
}
```

In this example:
- We define a static class `StringExtensions`.
- Inside this class, we define a static method `ToTitleCase` with the `this string str` parameter, indicating it's an extension method for the `string` type.
- The method capitalizes the first letter of each word in the string.

### Usage

Once defined, extension methods can be used as if they were instance methods of the extended type:

```csharp
string input = "hello world";
string result = input.ToTitleCase(); // Result will be "Hello World"
```

### Guidelines

Here are some guidelines for using extension methods effectively:
- Extension methods must be defined in a `static` class.
- The first parameter of an extension method specifies the type being extended and must be prefixed with `this`.
- Extension methods can be called directly on instances of the extended type, just like instance methods.
- They can be used to "add" methods to existing classes or structs, including types that you do not have control over (e.g., types from third-party libraries).

### Considerations

- Extension methods should be used sparingly and only when they provide clear utility or enhance readability without causing confusion.
- Avoid defining extension methods that could potentially conflict with existing methods or cause ambiguity.

Extension methods are a powerful feature in C# that enables you to augment existing types with additional functionality without modifying them, making your code more modular and readable.