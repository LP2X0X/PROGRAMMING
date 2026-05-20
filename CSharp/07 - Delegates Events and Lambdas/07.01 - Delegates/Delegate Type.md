---
tags:
 - csharp
 - delegates
---

# Delegate Type

## What Is a Delegate?

A delegate is a **type-safe object that points to one or more methods**, which can be invoked later. It's the .NET way of doing callbacks — in a type-safe, object-oriented manner.

A delegate holds three pieces of information:
1. The **address** of the method it points to
2. The **parameters** of that method
3. The **return type** of that method

Delegates can point to both **static** and **instance** methods.


---

## Defining a Delegate Type

Use the `delegate` keyword. The signature must match the method(s) it will point to:

```csharp
public delegate int BinaryOp(int x, int y);
```

This says: `BinaryOp` can point to **any method** that takes two `int`s and returns an `int`.


---

## What the Compiler Actually Generates

When you write `delegate`, the compiler generates a **sealed class** that inherits from `System.MulticastDelegate`. For the example above, it roughly produces:

```
BinaryOp : MulticastDelegate
├── Invoke(int x, int y) → int        // synchronous call
├── BeginInvoke(int x, int y, ...)     // async (legacy, pre-Task)
└── EndInvoke(IAsyncResult)            // async (legacy, pre-Task)
```

`Invoke()` is the key method — it calls the target method synchronously (the caller waits until it completes).


---

## Using a Delegate

```csharp
public delegate int BinaryOp(int x, int y);

static int Add(int a, int b) => a + b;
static int Multiply(int a, int b) => a * b;

// Create delegate instance pointing to Add
BinaryOp op = new BinaryOp(Add);
int result = op(3, 4); // 7 — shorthand for op.Invoke(3, 4)

// Reassign to a different method
op = Multiply;
result = op(3, 4); // 12
```

You can use the shorthand `op(3, 4)` — the compiler translates it to `op.Invoke(3, 4)`.


---

## Multicast Delegates

Since delegates inherit from `MulticastDelegate`, they can hold a **list of methods** using `+=`:

```csharp
public delegate void NotifyOp(string msg);

static void LogToConsole(string msg) => Console.WriteLine($"Console: {msg}");
static void LogToFile(string msg) => File.AppendAllText("log.txt", msg);

NotifyOp notify = LogToConsole;
notify += LogToFile; // now points to both methods

notify("Server started"); // both methods are called, in order
```

- **`+=`** adds a method to the invocation list.
- **`-=`** removes a method from the list.
- If the delegate has a return type, only the **last** method's return value is captured — earlier return values are discarded.


---

## Common Built-in Delegate Types

Instead of defining your own delegate types, you can often use the generic ones provided by .NET:

| Delegate | Signature | Use case |
|---|---|---|
| `Action` | `void ()` | No params, no return |
| `Action<T>` | `void (T)` | Takes param(s), no return |
| `Func<TResult>` | `TResult ()` | No params, returns a value |
| `Func<T, TResult>` | `TResult (T)` | Takes param(s), returns a value |
| `Predicate<T>` | `bool (T)` | Takes a param, returns bool |

```csharp
// Instead of: public delegate int BinaryOp(int x, int y);
Func<int, int, int> op = Add;
int result = op(3, 4); // 7
```

You only need a custom `delegate` when you want a meaningful name for readability or need `ref`/`out` parameters.
