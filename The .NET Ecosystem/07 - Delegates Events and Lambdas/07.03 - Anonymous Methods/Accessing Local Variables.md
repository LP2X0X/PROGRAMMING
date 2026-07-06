---
tags:
 - csharp
 - delegates
 - anonymous-methods
---

# Accessing Local Variables (Outer Variables)

## What Are Outer Variables?

An anonymous method can access local variables from the method that defines it. These are called **outer variables**. When an anonymous method uses an outer variable, it **captures** it — meaning the anonymous method holds a reference to that variable, not a copy of its value.

```csharp
int counter = 0;

Action increment = delegate {
    counter++;  // captures the local variable 'counter'
};

increment();
increment();
Console.WriteLine(counter); // 2 — the original variable was modified
```

The anonymous method and the enclosing method share the **same** `counter` variable. Changes inside the anonymous method are visible outside, and vice versa.


---

## How Capture Works Under the Hood

When an anonymous method captures a local variable, the compiler generates a **hidden class** (called a closure) that holds the captured variable as a field. Both the enclosing method and the anonymous method access the variable through this class:

```csharp
// What you write:
int counter = 0;
Action increment = delegate { counter++; };

// What the compiler roughly generates:
class <>DisplayClass
{
    public int counter;

    public void AnonymousMethod()
    {
        counter++;
    }
}

var closure = new <>DisplayClass();
closure.counter = 0;
Action increment = closure.AnonymousMethod;
```

This is why the variable is shared, not copied — both sides point to the same field on the same object.


---

## Scoping Rules

There are specific rules about what an anonymous method can and cannot access from its enclosing scope:

### Can Access

1. **Local variables** of the enclosing method (these become captured/outer variables)
2. **Instance variables** of the enclosing class (via the implicit `this` reference)
3. **Static variables** of the enclosing class
4. **Method parameters** of the enclosing method (regular value/reference parameters)

```csharp
class Example
{
    private int instanceVar = 10;
    private static int staticVar = 20;

    public void Demo(int param)
    {
        int local = 30;

        Action show = delegate {
            Console.WriteLine(instanceVar);  // instance variable — OK
            Console.WriteLine(staticVar);    // static variable — OK
            Console.WriteLine(param);        // method parameter — OK
            Console.WriteLine(local);        // local variable — OK (captured)
        };

        show();
    }
}
```

### Cannot Access

1. **`ref` or `out` parameters** of the enclosing method — these have special lifetime rules that conflict with capture.

```csharp
public void Demo(ref int x)
{
    // COMPILE ERROR: cannot capture ref parameter
    Action bad = delegate { Console.WriteLine(x); };
}
```

2. **A local variable with the same name as an outer local** — the scopes overlap and the compiler disallows the ambiguity.

```csharp
public void Demo()
{
    int value = 10;

    // COMPILE ERROR: 'value' already declared in the enclosing scope
    Action bad = delegate { int value = 20; };
}
```

3. However, a local variable **can** share a name with a **class member** — the local hides the member (same as normal C# shadowing rules).

```csharp
class Example
{
    private int value = 10;

    public void Demo()
    {
        // This 'value' hides the instance field — valid but potentially confusing
        Action show = delegate { int value = 20; Console.WriteLine(value); };
        show(); // 20
    }
}
```


---

## Watch Out: Captured Variable Lifetime

Because captured variables live on the heap (inside the compiler-generated closure class), they survive beyond the enclosing method's return. This is powerful but can cause surprises:

```csharp
static Func<int> CreateCounter()
{
    int count = 0;
    return delegate { return ++count; };
}

var counter = CreateCounter();
Console.WriteLine(counter()); // 1
Console.WriteLine(counter()); // 2
Console.WriteLine(counter()); // 3
// 'count' lives on even though CreateCounter() has returned
```

### The Classic Loop Trap

A common mistake — capturing a loop variable:

```csharp
var actions = new List<Action>();

for (int i = 0; i < 3; i++)
{
    actions.Add(delegate { Console.WriteLine(i); });
}

foreach (var action in actions)
    action();

// Output: 3, 3, 3 — NOT 0, 1, 2!
// All three delegates captured the SAME variable 'i',
// which is 3 by the time the loop finishes.
```

**Fix** — capture a copy inside the loop:

```csharp
for (int i = 0; i < 3; i++)
{
    int copy = i;  // each iteration gets its own 'copy'
    actions.Add(delegate { Console.WriteLine(copy); });
}
// Output: 0, 1, 2 ✓
```

```ad-note
`foreach` loops in C# 5.0+ do **not** have this problem — each iteration gets its own copy of the loop variable. The trap only applies to `for` loops.
```
