---
tags:
 - csharp
 - delegates
 - events
 - lambda
---

## 1. Delegate — Defines the Method Signature

A delegate is a **type-safe function pointer**. It defines what kind of method (return type + parameters) can be assigned to it.

```csharp
public delegate void MyDelegate(string message);
```

This means: "any method that returns `void` and accepts a single `string` can be used with this delegate."

A delegate is a **type** (like a class). An instance of it holds a reference to one or more methods.

```csharp
MyDelegate del = SomeMethod;        // holds one method
del += AnotherMethod;               // now holds two (multicast)
del.Invoke("hello");                // calls both, in order
```

---

## 2. Event — A Restricted Delegate

An event is a delegate with **encapsulation enforced by the compiler**. Outside code can only `+=` and `-=` — it cannot assign (`=`), invoke, or wipe subscribers.

```csharp
public event MyDelegate MyEvent;
```

| | Public delegate field | `event` |
|---|---|---|
| `+=` / `-=` from outside | Yes | Yes |
| `=` (overwrite) from outside | Yes — dangerous | Compile error |
| Invoke from outside | Yes — dangerous | Compile error |

Inside the declaring class, the event behaves like a normal delegate. Outside, it's locked down. That's the entire purpose of `event`.

---

## 3. Event Handlers — Must Match the Delegate Signature

Subscribed methods (event handlers / callbacks) must match the delegate's signature. They can be attached as named methods, anonymous methods, or lambdas:

```csharp
// Named method
void HandleEvent(string msg) => Console.WriteLine(msg);
MyEvent += HandleEvent;

// Anonymous method (C# 2.0 syntax)
MyEvent += delegate(string msg) { Console.WriteLine(msg); };

// Lambda (C# 3.0+ syntax)
MyEvent += (msg) => Console.WriteLine(msg);
```

All three produce a delegate instance pointing to a function. The lambda is just the most concise syntax.

---

## 4. Invoke — Fires the Event

When invoked, the event calls **all subscribers** in the order they were added:

```csharp
MyEvent?.Invoke("Hello Events!");
```

`?.Invoke` is used because if no handlers are subscribed, the event is `null`.

---

## 5. The Standard .NET Event Pattern

.NET provides built-in generic delegates so you don't have to define your own:

| Delegate | Signature | Use when |
|---|---|---|
| `EventHandler` | `void(object sender, EventArgs e)` | Event carries no extra data |
| `EventHandler<TEventArgs>` | `void(object sender, TEventArgs e)` | Event carries custom data |

`EventHandler<TEventArgs>` is shorthand for declaring a custom delegate:

```csharp
// Without EventHandler<T> — manual delegate
public delegate void CarExplodedHandler(object sender, CarEventArgs e);
public event CarExplodedHandler Exploded;

// With EventHandler<T> — no custom delegate needed
public event EventHandler<CarEventArgs> Exploded;
```

Same signature, less boilerplate. For events with no data, use `EventHandler` and pass `EventArgs.Empty`.

---

## 6. The Connection Between Lambdas and Delegates

A lambda **is** a function (the code/logic), but in C# it can't exist on its own — it always needs a **delegate type** to hold it:

- **Delegate** = the type (the container)
- **Lambda** = the function body (the value)

```csharp
Func<int, bool> isPositive = (n) => n > 0;
//   ↑ delegate (type)        ↑ lambda (function body)
```

A lambda can only appear where the compiler knows what delegate type to assign it to:

```csharp
Func<int, bool> f = (n) => n > 0;                        // target type: Func<int,bool>
Run((msg) => Console.WriteLine(msg));                     // target type: Action<string>
c1.AboutToBlow += (sender, e) => Console.WriteLine("!"); // target type: EventHandler<CarEventArgs>
```

> [!important]
> The `=>` in a lambda and the `=>` in an expression-bodied member are **different things**:
> - Lambda: `(n) => n > 0` — creates a delegate instance
> - Expression-bodied member: `public int Double(int n) => n * 2;` — just a syntax shortcut for `{ return ...; }`, no delegate involved

---

## 7. Evolution of Syntax

All of these create the same delegate instance — each version is a shorter way to write the previous:

```csharp
// 1. Named method + explicit delegate construction
bool IsPositive(int n) { return n > 0; }
Func<int, bool> a = new Func<int, bool>(IsPositive);

// 2. Anonymous method (C# 2.0) — inline the body
Func<int, bool> b = delegate(int n) { return n > 0; };

// 3. Lambda expression (C# 3.0) — shortest form
Func<int, bool> c = (n) => n > 0;
```

---

## Conceptual View

```
Delegate         ← defines the method signature (the type)
   ↓
Event            ← wraps a delegate with encapsulation (only +=/-= from outside)
   ↓
Subscribe (+=)   ← attach handlers (named method, anonymous method, or lambda)
   ↓
Invoke           ← fires the event, calling all subscribers in order
```

> "An event uses `Invoke()` to notify all its subscribed event handlers (callbacks). These handlers must match the method signature defined by a delegate, ensuring type safety and consistency."
