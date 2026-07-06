---
tags:
 - csharp
 - events
 - generics
---

## What Is `EventHandler<TEventArgs>`?

`EventHandler<TEventArgs>` is a **built-in generic delegate** provided by .NET. It saves you from having to declare a custom delegate every time you create an event that carries data.

Its definition in the framework is simply:

```csharp
public delegate void EventHandler<TEventArgs>(object? sender, TEventArgs e);
```

So when you write:

```csharp
public event EventHandler<CarEventArgs> Exploded;
```

That's equivalent to manually defining a delegate and using it:

```csharp
// Without EventHandler<T> — you define your own delegate
public delegate void CarExplodedHandler(object? sender, CarEventArgs e);
public event CarExplodedHandler Exploded;
```

`EventHandler<TEventArgs>` just removes the need for that first line. You get the standard .NET event signature `(object sender, TEventArgs e)` automatically.

---

## Why It Exists

Before generics (.NET 1.x), every event with custom data needed its own delegate:

```csharp
public delegate void CarExplodedHandler(object sender, CarEventArgs e);
public delegate void CarStartedHandler(object sender, CarStartedEventArgs e);
public delegate void CarStoppedHandler(object sender, CarStoppedEventArgs e);
// ... one delegate per event
```

With `EventHandler<T>`, you don't declare any of them:

```csharp
public event EventHandler<CarEventArgs> Exploded;
public event EventHandler<CarStartedEventArgs> Started;
public event EventHandler<CarStoppedEventArgs> Stopped;
```

Same signatures, zero custom delegates.

---

## `EventHandler` vs `EventHandler<T>`

| | `EventHandler` | `EventHandler<TEventArgs>` |
|---|---|---|
| Signature | `void(object sender, EventArgs e)` | `void(object sender, TEventArgs e)` |
| Carries custom data | No — just `EventArgs.Empty` | Yes — your `EventArgs` subclass |
| Use when | Event has no extra info to pass | Event needs to send data to subscribers |

```csharp
// No data — use non-generic EventHandler
public event EventHandler DoorOpened;

// Raising it:
DoorOpened?.Invoke(this, EventArgs.Empty);

// With data — use generic EventHandler<T>
public event EventHandler<CarEventArgs> Exploded;

// Raising it:
Exploded?.Invoke(this, new CarEventArgs("Engine overheated"));
```

---

## Full Example

```csharp
public class CarEventArgs : EventArgs
{
    public string Message { get; }
    public CarEventArgs(string message) => Message = message;
}

public class Car
{
    public event EventHandler<CarEventArgs>? Exploded;

    private int _speed;

    public void Accelerate()
    {
        _speed += 20;
        if (_speed > 100)
        {
            OnExploded(new CarEventArgs($"Engine blew at {_speed} km/h"));
        }
    }

    protected virtual void OnExploded(CarEventArgs e)
    {
        Exploded?.Invoke(this, e);
    }
}

// Subscriber
var car = new Car();
car.Exploded += (sender, e) => Console.WriteLine(e.Message);

for (int i = 0; i < 10; i++)
    car.Accelerate();

// Output: Engine blew at 120 km/h
//         Engine blew at 140 km/h
//         ... etc.
```

> [!tip]
> You almost never need to declare your own event delegate in modern .NET. Use `EventHandler` for no-data events and `EventHandler<T>` for everything else.
