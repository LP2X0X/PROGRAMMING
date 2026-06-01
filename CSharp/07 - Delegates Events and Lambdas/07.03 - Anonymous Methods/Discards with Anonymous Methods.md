---
tags:
 - csharp
 - delegates
 - anonymous-methods
---

# Discards with Anonymous Methods

## What Are Discards?

A discard (`_`) is a parameter you **intentionally don't use**. It tells the compiler and anyone reading the code: "this parameter is required by the delegate signature but irrelevant to this handler."

```csharp
// Without discards — sender and e are named but never used
c1.AboutToBlow += delegate(object sender, CarEventArgs e)
{
    Console.WriteLine("Something is about to blow!");
};

// With discards — clearly signals these parameters are not needed
c1.AboutToBlow += delegate(object _, CarEventArgs _)
{
    Console.WriteLine("Something is about to blow!");
};

// Same idea with lambda syntax
c1.AboutToBlow += (_, _) => Console.WriteLine("Something is about to blow!");
```


---

## The Two-Discard Rule

```ad-warning
Because `_` was a legal variable name in older C# versions, there must be **two or more** discards for the compiler to treat them as true discards. A single `_` is treated as a normal variable name for backward compatibility.
```

```csharp
// Two discards — both treated as discards ✓
Func<int, int, int> constant = delegate (int _, int _) { return 42; };
Console.WriteLine(constant(3, 4)); // 42

// Single _ — NOT a discard, it's a regular parameter named '_'
Func<int, int> identity = delegate (int _) { return _; };
Console.WriteLine(identity(7)); // 7 — _ is used as a variable
```

This means with a single-parameter delegate, you **cannot** use `_` as a discard — it will always be a variable. In that case, use a descriptive but clearly-unused name, or just ignore it:

```csharp
// Single-parameter delegate — _ is a variable, not a discard
Action<int> handler = delegate (int _) {
    // _ is accessible here as a regular int parameter
    Console.WriteLine("Handled");
};
```


---

## When to Use Discards

Use discards when the delegate signature **forces** you to accept parameters you don't need. Most common with events:

```csharp
// Don't care about sender or args
button.Click += (_, _) => Console.WriteLine("Clicked!");

// Care about args but not sender
button.Click += (_, e) => HandleClick(e);

// Care about sender but not args
button.Click += (sender, _) => Log($"Clicked by {sender}");
```

Discards make the **intent** explicit — a named parameter that's never used looks like a bug. A discard says "I know this exists, I don't need it."
