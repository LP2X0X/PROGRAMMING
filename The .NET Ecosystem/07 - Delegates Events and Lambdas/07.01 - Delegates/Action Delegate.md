---
tags:
 - csharp
 - delegates
---

# Action Delegate

A built-in generic delegate that points to a method returning `void`. Exists in overloads from 0 to 16 parameters:

```csharp
Action                        // void ()
Action<T>                     // void (T)
Action<T1, T2>                // void (T1, T2)
// ... up to Action<T1, ..., T16>
```

The `in` keyword on type parameters means they are **contravariant** — you can assign an `Action<object>` to an `Action<string>`.


---

## Basic Usage

```csharp
// No parameters
Action greet = () => Console.WriteLine("Hello!");
greet(); // Hello!

// One parameter
Action<string> log = msg => Console.WriteLine($"[LOG] {msg}");
log("Server started"); // [LOG] Server started

// Multiple parameters
Action<string, int> repeat = (text, count) =>
{
    for (int i = 0; i < count; i++)
        Console.Write(text);
    Console.WriteLine();
};
repeat("Ha", 3); // HaHaHa
```


---

## Passing Action as a Method Parameter

This is the main reason `Action` exists — passing behavior into a method without defining a custom delegate:

```csharp
static void ProcessItems(List<string> items, Action<string> process)
{
    foreach (var item in items)
        process(item);
}

var names = new List<string> { "Alice", "Bob", "Charlie" };

// Pass different behaviors
ProcessItems(names, name => Console.WriteLine(name.ToUpper()));
ProcessItems(names, name => File.AppendAllText("log.txt", name + "\n"));
```


---

## Pointing to Named Methods

`Action` doesn't have to use lambdas — it can point to any matching method:

```csharp
static void PrintToConsole(string msg) => Console.WriteLine(msg);

Action<string> print = PrintToConsole;
print("test"); // test
```


---

## Multicast (Multiple Targets)

Like all delegates, `Action` supports multicast via `+=`:

```csharp
Action<string> notify = msg => Console.WriteLine($"Email: {msg}");
notify += msg => Console.WriteLine($"SMS: {msg}");
notify += msg => Console.WriteLine($"Push: {msg}");

notify("Order shipped");
// Email: Order shipped
// SMS: Order shipped
// Push: Order shipped
```


---

## Real-World Use Cases

**Callback after async work:**
```csharp
static void FetchData(string url, Action<string> onComplete)
{
    string data = httpClient.GetStringAsync(url).Result;
    onComplete(data);
}

FetchData("https://api.example.com", json => Console.WriteLine(json));
```

**Configuring behavior in a method (strategy pattern):**
```csharp
static void SortAndDisplay(List<int> list, Action<List<int>> sortStrategy)
{
    sortStrategy(list);
    list.ForEach(Console.WriteLine);
}

SortAndDisplay(numbers, l => l.Sort());                    // ascending
SortAndDisplay(numbers, l => l.Sort((a, b) => b - a));     // descending
```

**List.ForEach — Action in the standard library:**
```csharp
var names = new List<string> { "Alice", "Bob" };
names.ForEach(name => Console.WriteLine(name));
// List<T>.ForEach signature: void ForEach(Action<T> action)
```


---

## Action vs Func

| | `Action` | `Func` |
|---|---|---|
| Returns | `void` | a value (`TResult`) |
| Use when | performing a side effect (log, write, modify) | computing and returning a result |

```csharp
Action<int> print   = x => Console.WriteLine(x);  // no return
Func<int, int> square = x => x * x;               // returns int
```
