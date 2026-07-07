---
tags:
 - csharp
 - interfaces
 - ienumerable
---

# Yield and State Machine

`yield return` is syntactic sugar — the compiler rewrites your method into a **state machine class** that implements `IEnumerable<T>` and `IEnumerator<T>`.

---

## What You Write

```csharp
public static IEnumerable<int> GetNumbers()
{
    yield return 1;
    yield return 2;
    yield return 3;
}
```

## What the Compiler Generates (Simplified)

```csharp
class <GetNumbers>d__0 : IEnumerable<int>, IEnumerator<int>
{
    private int _state;
    private int _current;

    public bool MoveNext()
    {
        switch (_state)
        {
            case 0:
                _current = 1;
                _state = 1;
                return true;
            case 1:
                _current = 2;
                _state = 2;
                return true;
            case 2:
                _current = 3;
                _state = -1;
                return true;
            default:
                return false;
        }
    }

    public int Current => _current;
}
```

Each `yield return` becomes a state in the switch. When the caller calls `MoveNext()`, execution **resumes from where it left off** — the method appears to pause and continue, but it's really the state machine advancing to the next case.

---

## Why This Matters

- **Lazy evaluation** — items are produced one at a time, only when requested. Nothing runs until someone iterates.
- **No intermediate collections** — unlike returning a `List<T>`, yield doesn't allocate a full collection in memory.
- **Short-circuiting** — consumers like `First()` or `Take(5)` can stop early without producing all items.

```csharp
public static IEnumerable<int> EvenNumbers()
{
    int n = 0;
    while (true)    // infinite — only possible because of lazy evaluation
    {
        yield return n;
        n += 2;
    }
}

// Only produces 5 items, then stops
var first5 = EvenNumbers().Take(5);  // 0, 2, 4, 6, 8
```

---

## yield break

Use `yield break` to end the sequence early:

```csharp
public static IEnumerable<int> GetUntilNegative(int[] numbers)
{
    foreach (var n in numbers)
    {
        if (n < 0)
            yield break;    // sets _state to -1, MoveNext returns false

        yield return n;
    }
}
```

---

## Key Points

|                          | Detail                                                                                  |
| ------------------------ | --------------------------------------------------------------------------------------- |
| What `yield return` does | Saves the current position, returns one item, resumes on next `MoveNext()`              |
| What `yield break` does  | Terminates the sequence — no more items                                                 |
| Compiler output          | A hidden class implementing `IEnumerable<T>` + `IEnumerator<T>` with a state machine    |
| Why it's syntactic sugar | You could write the state machine by hand — `yield` just makes the compiler do it       |
| Execution model          | Deferred — nothing runs until someone iterates (e.g., `foreach`, `ToList()`, `First()`) | 
