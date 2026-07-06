---
tags:
 - csharp
 - events
---

# Events

## The Problem: Why Not Just Use a Public Delegate?

You *could* use a plain public delegate field to let outside code register callbacks:

```csharp
public class Button
{
    public Action? Clicked; // just a public delegate
}

var btn = new Button();
btn.Clicked += () => Console.WriteLine("handled");
```

This works, but it has **dangerous flaws**:

```csharp
// Any outside code can wipe all subscribers
btn.Clicked = null;

// Any outside code can invoke the delegate directly
btn.Clicked();

// Any outside code can replace with = instead of +=, silently removing other subscribers
btn.Clicked = () => Console.WriteLine("I replaced everyone");
```

A public delegate gives callers **full control** — they can assign, invoke, and overwrite it. The publisher has no protection over its own notification mechanism.


---

## The Solution: The `event` Keyword

`event` is a **restriction layer** on top of a delegate. It enforces that outside code can **only** `+=` and `-=`:

```csharp
public class Button
{
    public event Action? Clicked; // event — not a plain delegate
}

var btn = new Button();
btn.Clicked += () => Console.WriteLine("handled");  // OK
btn.Clicked -= handler;                              // OK

btn.Clicked = null;    // COMPILE ERROR — can't assign from outside
btn.Clicked();         // COMPILE ERROR — can't invoke from outside
```

**Inside** the declaring class, the event behaves like a normal delegate — you can assign and invoke it. **Outside**, it's locked down to subscribe/unsubscribe only.

That's the entire purpose of `event`: **encapsulation of the delegate**.


---

## Publisher / Subscriber Pattern

- **Publisher** — the class that declares the event and decides when to raise it.
- **Subscriber** — any class that attaches a handler via `+=`.
- The publisher doesn't know or care who's listening. Subscribers don't need to know about each other.

```
Publisher (raises event)  →  Event  ←  Subscriber A (+=)
                                    ←  Subscriber B (+=)
                                    ←  Subscriber C (+=)
```


---

## Full Example

```csharp
public class Counter
{
    private int count;

    // 1. Declare the event
    public event EventHandler? ThresholdReached;

    public void CountUp()
    {
        count++;
        Console.WriteLine($"Count: {count}");

        if (count == 5)
            OnThresholdReached(EventArgs.Empty); // 2. Raise it
    }

    // 3. Protected virtual method — standard pattern
    //    Lets derived classes override the raising behavior
    protected virtual void OnThresholdReached(EventArgs e)
    {
        ThresholdReached?.Invoke(this, e);
    }
}

// Subscriber
var counter = new Counter();
counter.ThresholdReached += (sender, e) => Console.WriteLine("Threshold hit!");

for (int i = 0; i < 10; i++)
    counter.CountUp();

// Output:
// Count: 1
// Count: 2
// ...
// Count: 5
// Threshold hit!
// Count: 6
// ...
```


---

## The Standard Event Pattern

.NET has a convention for events:

1. **Delegate type:** use `EventHandler` (no data) or `EventHandler<TEventArgs>` (with data).
2. **Signature:** `void Handler(object? sender, EventArgs e)` — sender is the object that raised the event.
3. **Raising method:** `protected virtual void OnEventName(EventArgs e)` — lets subclasses override.

### Custom Event Data

If you need to pass data with the event, create a class inheriting `EventArgs`:

```csharp
public class ThresholdReachedEventArgs : EventArgs
{
    public int Threshold { get; }
    public DateTime TimeReached { get; }

    public ThresholdReachedEventArgs(int threshold, DateTime timeReached)
    {
        Threshold = threshold;
        TimeReached = timeReached;
    }
}

public class Counter
{
    private int count;

    public event EventHandler<ThresholdReachedEventArgs>? ThresholdReached;

    public void CountUp()
    {
        count++;
        if (count == 5)
        {
            var args = new ThresholdReachedEventArgs(5, DateTime.Now);
            OnThresholdReached(args);
        }
    }

    protected virtual void OnThresholdReached(ThresholdReachedEventArgs e)
    {
        ThresholdReached?.Invoke(this, e);
    }
}

// Subscriber can now access the data
counter.ThresholdReached += (sender, e) =>
    Console.WriteLine($"Threshold {e.Threshold} reached at {e.TimeReached}");
```


---

## Unsubscribing

Always unsubscribe when the subscriber's lifetime is shorter than the publisher's — otherwise the publisher holds a reference to the subscriber, preventing it from being garbage collected (**memory leak**):

```csharp
EventHandler handler = (s, e) => Console.WriteLine("handled");

counter.ThresholdReached += handler;   // subscribe
counter.ThresholdReached -= handler;   // unsubscribe — must be the same delegate instance
```

Anonymous lambdas can't be unsubscribed (you don't have a reference to pass to `-=`). If you need to unsubscribe, store the delegate in a variable first.


---

## What `event` Compiles To

Under the hood, the compiler generates:

```csharp
// What you write:
public event EventHandler? ThresholdReached;

// What the compiler generates:
private EventHandler? _thresholdReached;                        // backing field
public void add_ThresholdReached(EventHandler value) { ... }    // += calls this
public void remove_ThresholdReached(EventHandler value) { ... } // -= calls this
```

The backing delegate is **private**. The `add`/`remove` accessors are public. That's how encapsulation is enforced.


---

## Summary: event vs Plain Delegate

| | Public delegate field | `event` |
|---|---|---|
| `+=` / `-=` from outside | Yes | Yes |
| `=` (overwrite) from outside | Yes — **dangerous** | No — compile error |
| Invoke from outside | Yes — **dangerous** | No — compile error |
| Invoke from declaring class | Yes | Yes |
| Purpose | General callback | Encapsulated notification |
