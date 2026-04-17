---
tags:
 - csharp
 - oop
 - keyword
---

A C# class may define any number of static members, which are declared using the `static` keyword. When you do so, the member in question must be invoked directly from the class level, rather than from an object reference variable.

```ad-note
While any class can define static members, they are quite commonly found within utility classes.
```

## Static Members

Static members belong to the **class itself**, not to any particular instance. There is only one copy shared across all instances.

```csharp
class BankAccount
{
    public static double InterestRate = 0.03;
    public double Balance { get; set; }
}

// accessed via class name, not an instance
double rate = BankAccount.InterestRate;
```

You can apply `static` to: fields, properties, methods, events, operators, and constructors.

## Static Methods

Static methods can only access other static members directly — they have no access to `this` or instance members.

```csharp
class MathHelper
{
    public static int Add(int a, int b) => a + b;
}

int sum = MathHelper.Add(3, 5);
```

## Static Classes

A class marked `static` **cannot be instantiated or inherited**. All of its members must also be static. Commonly used for utility/helper classes.

```csharp
static class StringUtils
{
    public static bool IsNullOrEmpty(string s) => s == null || s.Length == 0;
}
```

## Static Constructors

A static constructor runs **once**, automatically, before the class is first used. It cannot take parameters or have access modifiers. This allows you to safely set the values of your static data.

```ad-note
The static constructor can not be overloaded.
```

```csharp
class Logger
{
    static string _logPath;

    static Logger()
    {
        _logPath = "app.log"; // one-time initialization
    }
}
```

## Static Local Functions

Local functions inside a method can be marked `static` to prevent them from capturing any enclosing variables (available since C# 8).

```csharp
void Process(int[] items)
{
    int sum = Calculate(items);

    static int Calculate(int[] data) // cannot capture 'sum' or other locals
    {
        return data.Sum();
    }
}
```

## Key Points

- Static members are allocated once and shared across all instances
- Static classes cannot be instantiated (`new`) or used as a base class
- Static constructors are parameterless, have no access modifier, and run at most once
- A non-static class can contain both static and instance members
- Static members are accessed via `ClassName.Member`, never through an instance