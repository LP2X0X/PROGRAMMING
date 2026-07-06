---
tags:
 - csharp
 - delegates
 - anonymous-methods
---

# Anonymous Methods

## What Is an Anonymous Method?

An anonymous method is an **inline, unnamed method** defined using the `delegate` keyword. Instead of declaring a separate named method and assigning it to a delegate, you write the method body right where you need it.

```ad-attention
Anonymous methods are shorthand for allocating a raw delegate and manually building a delegate target method. The compiler generates a hidden method behind the scenes.
```


---

## Syntax

```csharp
delegate (parameters) {
    // method body
};
```

- `delegate` keyword introduces the anonymous method.
- `(parameters)` — the parameters it accepts (must match the delegate signature).
- `{ }` — the method body.

```ad-danger
The closing brace must be followed by a semicolon. Forgetting it causes a compilation error.
```


---

## Basic Example

```csharp
// Named method approach
static void Greet(string name) => Console.WriteLine($"Hello, {name}!");

Action<string> greetDel = Greet;
greetDel("Alice"); // Hello, Alice!

// Anonymous method approach — no separate method needed
Action<string> greetAnon = delegate (string name) {
    Console.WriteLine($"Hello, {name}!");
};

greetAnon("Alice"); // Hello, Alice!
```

Both do the same thing. The anonymous method eliminates the need for a standalone `Greet` method when it's only used in one place.


---

## Parameterless Anonymous Methods

If the delegate takes no parameters, you can omit the parameter list entirely:

```csharp
Action sayHello = delegate {
    Console.WriteLine("Hello!");
};

sayHello(); // Hello!
```

You can also omit the parameter list even when the delegate **does** have parameters — if you don't need them:

```csharp
// Button.Click expects (object sender, EventArgs e) but we don't care
button.Click += delegate {
    Console.WriteLine("Clicked!");
};
```

This is only possible with anonymous methods, not with lambda expressions (lambdas require you to declare the parameters).


---

## Anonymous Methods vs. Lambda Expressions

Anonymous methods came first (C# 2.0). Lambda expressions (C# 3.0) largely replaced them with cleaner syntax:

```csharp
// Anonymous method
Func<int, int, int> add1 = delegate (int a, int b) { return a + b; };

// Lambda expression — same thing, less noise
Func<int, int, int> add2 = (a, b) => a + b;
```

| Feature                      | Anonymous Method             | Lambda Expression  |
| ---------------------------- | ---------------------------- | ------------------ |
| Syntax                       | `delegate (params) { body }` | `(params) => body` |
| Omit parameter list entirely | Yes                          | No                 |
| Expression body (no braces)  | No                           | Yes                |
| Used in expression trees     | No                           | Yes                |
| Introduced in                | C# 2.0                       | C# 3.0             |

```ad-note
Lambda expressions are the preferred syntax in modern C#. Anonymous methods still appear in legacy code and in the rare case where you want to omit the parameter list entirely (the one thing lambdas can't do).
```


---

## Common Use Cases

**Event handlers** — quick inline handlers without a separate method:

```csharp
button.Click += delegate (object sender, EventArgs e) {
    Console.WriteLine("Button clicked!");
};
```

**Passing callbacks** — passing behavior to a method:

```csharp
List<int> numbers = [1, 2, 3, 4, 5];

List<int> evens = numbers.FindAll(delegate (int n) {
    return n % 2 == 0;
});
// evens: [2, 4]
```

**Thread/Task work** — inline work items:

```csharp
Thread t = new Thread(delegate () {
    Console.WriteLine("Running on a separate thread");
});
t.Start();
```


---

## What the Compiler Generates

When you write an anonymous method, the compiler generates a **hidden private method** inside the class. For example:

```csharp
Action<string> greet = delegate (string name) {
    Console.WriteLine($"Hello, {name}!");
};
```

Becomes roughly:

```csharp
// Compiler-generated (you never see this)
private void <>AnonymousMethod_0(string name)
{
    Console.WriteLine($"Hello, {name}!");
}

// Your delegate now points to this generated method
Action<string> greet = new Action<string>(<>AnonymousMethod_0);
```

If the anonymous method captures local variables (see [[Accessing Local Variables]]), the compiler generates a **hidden class** to hold those variables instead.
