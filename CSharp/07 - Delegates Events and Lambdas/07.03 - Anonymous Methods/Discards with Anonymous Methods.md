---
tags:
 - csharp
 - delegates
 - anonymous-methods
---

## What Are Discards in Anonymous Methods?

A discard (`_`) is a parameter you **intentionally don't use**. It tells the compiler and anyone reading the code: "this parameter is required by the delegate signature but irrelevant to this handler."

```csharp
// Without discards — sender and e are named but never used (misleading)
c1.AboutToBlow += delegate(object sender, CarEventArgs e)
{
    Console.WriteLine("Something is about to blow!");
};

// With discards — clearly signals these parameters are not needed
c1.AboutToBlow += delegate(object _, CarEventArgs _)
{
    Console.WriteLine("Something is about to blow!");
};

// Same with lambda syntax
c1.AboutToBlow += (_, _) => Console.WriteLine("Something is about to blow!");
```

---

## The Two-Discard Rule

```ad-note
Because the underscore (`_`) was a legal variable identifier in previous versions of C#, there must be **two or more** discards used with the anonymous method to be treated as discards. A single `_` is treated as a normal variable name for backward compatibility.
```

```csharp
// Two discards — both treated as discards ✓
Func<int, int, int> constant = delegate (int _, int _) { return 42; };
Console.WriteLine("constant(3,4)={0}", constant(3, 4));  // 42

// Single _ — treated as a normal parameter name, NOT a discard
Func<int, int> identity = delegate (int _) { return _; };  // _ is a variable here
```

---

## When to Use

Use discards when the delegate signature forces you to accept parameters you don't need. Common with events where you don't care about `sender` or the event args:

```csharp
// Don't care who sent it or what the args contain
button.Clicked += (_, _) => Console.WriteLine("Clicked!");

// Only care about the args, not the sender
button.Clicked += (_, e) => Console.WriteLine(e.Message);
```