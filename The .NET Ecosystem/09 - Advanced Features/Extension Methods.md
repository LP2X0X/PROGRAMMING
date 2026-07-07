---
tags:
 - csharp
 - advanced
---

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

---

## How It Actually Works

Extension methods are **purely a compiler trick**. There is nothing special at the IL level — the compiler rewrites the call into a regular static method call.

```csharp
// What you write:
string result = input.ToTitleCase();

// What the compiler emits:
string result = StringExtensions.ToTitleCase(input);
```

This means extension methods have **no access to private or protected members** of the extended type. They can only use the type's public API — exactly like any other external code.

---

## Rules

| Rule | Detail |
|---|---|
| Must be in a static class | The class itself must be `static` |
| Must be a static method | The method must be `static` |
| First parameter uses `this` | `this` before the type marks it as an extension |
| Must be in scope | The namespace containing the static class must be imported with `using` |
| Cannot be nested | The static class cannot be inside another class |

---

## Instance Methods Always Win

If the type already has an instance method with the same name and compatible signature, **the instance method is always called**. The extension method is silently ignored — no warning, no error.

```csharp
public class MyClass
{
    public void Print() => Console.WriteLine("Instance");
}

public static class MyClassExtensions
{
    public static void Print(this MyClass obj) => Console.WriteLine("Extension");
}

var x = new MyClass();
x.Print();  // "Instance" — the extension is never called
```

This means adding an instance method to a class in a future version **can silently break** code that relied on an extension method with the same name.

---

## Calling on Null

Since extension methods are static method calls, `this` can be `null`. This is different from instance methods which throw `NullReferenceException`.

```csharp
public static class StringExtensions
{
    public static bool IsNullOrEmpty(this string s) => string.IsNullOrEmpty(s);
}

string s = null;
s.IsNullOrEmpty();  // works — returns true, no NullReferenceException
```

This can be useful, but it can also surprise callers who expect `null.Method()` to always throw.

---

## Extending Interfaces

One of the most powerful uses — you can extend an interface, and **every type that implements it** gets the method.

```csharp
public static class EnumerableExtensions
{
    public static bool IsEmpty<T>(this IEnumerable<T> source) => !source.Any();
}

List<int> list = new();
int[] array = Array.Empty<int>();

list.IsEmpty();   // true — List<T> implements IEnumerable<T>
array.IsEmpty();  // true — arrays implement IEnumerable<T>
```

This is exactly how LINQ works. `Where`, `Select`, `OrderBy` etc. are all extension methods on `IEnumerable<T>` defined in `System.Linq.Enumerable`.

---

## Chaining

Because extension methods can return a value, they enable fluent-style chaining:

```csharp
public static class StringExtensions
{
    public static string RemoveSpaces(this string s) => s.Replace(" ", "");
    public static string AddPrefix(this string s, string prefix) => prefix + s;
}

string result = "hello world"
    .RemoveSpaces()
    .AddPrefix(">>")
    .ToUpper();
// result: ">>HELLOWORLD"
```

Each method returns a `string`, so the next extension method can be called on the result. This is the same pattern LINQ uses to chain `Where().Select().OrderBy()`.

---

## Generic Extension Methods

Extension methods can be generic, making them reusable across many types:

```csharp
public static class ObjectExtensions
{
    public static bool IsIn<T>(this T item, params T[] collection)
        => collection.Contains(item);
}

int x = 5;
x.IsIn(1, 3, 5, 7);     // true

string s = "hello";
s.IsIn("hi", "hello");   // true
```

---

## When to Use Extension Methods

**Good use cases:**
- Adding utility methods to types you don't own (framework types, third-party libraries)
- Extending interfaces to give default behavior to all implementors
- Building fluent APIs and method chains
- Keeping a type's own class lean while offering optional convenience methods in a separate namespace

**Avoid when:**
- You own the type and can just add the method directly
- The method needs access to private state — it can't, so you'd be fighting the design
- The method name could collide with a future instance method on the type

```ad-warning
Extension methods can make code harder to navigate — the method looks like it belongs to the type but lives in a completely different class. Use namespaces intentionally: put extension methods in a namespace the consumer must explicitly import, so they opt in.
```