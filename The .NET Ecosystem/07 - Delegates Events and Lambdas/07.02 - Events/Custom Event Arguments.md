---
tags:
 - csharp
 - events
---

In C#, when you define custom events, you often want to pass additional information about the event to the event handlers. This is where custom event arguments come into play. Custom event arguments allow you to create a class that inherits from `EventArgs` to encapsulate any necessary data related to the event.

### Why Use the `EventHandler`/`EventArgs` Pattern at All?

The standard .NET event pattern `(object sender, EventArgs e)` is a **convention, not a language requirement**. You can use any delegate for events — for example, a simple `Action<string>` or a custom delegate like `delegate void MyHandler(string message)` would compile and work fine.

However, the standard pattern exists for good reasons:

- **Consistency** — every event across .NET libraries follows the same shape: `(object sender, EventArgs e)`. Consumers always know what to expect.
- **Extensibility** — if you later need to pass more data, you subclass `EventArgs` and add properties, instead of changing the delegate signature which would **break all existing subscribers**.

For example, with a custom delegate `CarEngineHandler(string message)`, if you later want to also pass `Speed`, you'd have to change the signature to `CarEngineHandler(string message, int speed)` and update every subscriber. With the standard pattern, you just add a property to your `EventArgs` subclass — existing subscribers still compile and just ignore the new field.

### `EventArgs.Empty`

When raising an event that carries **no extra data**, use `EventArgs.Empty` instead of `new EventArgs()`:

```csharp
SomeEvent?.Invoke(this, EventArgs.Empty);  // reuses a single pre-allocated instance
SomeEvent?.Invoke(this, new EventArgs());  // allocates a new object each time (unnecessary)
```

`EventArgs.Empty` is a static, pre-allocated instance. Since `EventArgs` has no mutable state, all instances are identical — there's no reason to create a new one each time. Same idea as `string.Empty`. It is **not** implicitly called anywhere; you must always pass it explicitly.

### Steps to Implement Custom Event Arguments

1. **Define Custom Event Arguments Class**:
   - Create a class that inherits from `EventArgs`. This class will hold properties and fields to convey information specific to your event.

2. **Define the Event in the Publisher Class**:
   - Declare an event in your publisher class using the delegate type that includes your custom event arguments.

3. **Raise the Event with Custom Event Arguments**:
   - When raising the event (invoking the delegate), create an instance of your custom event arguments class and pass it to the event handler.

4. **Handle the Event in Subscribers**:
   - Define event handler methods in subscriber classes that match the delegate signature and handle the custom event arguments.

### Example of Custom Event Arguments

Let’s illustrate with an example where a `TemperatureSensor` class raises an event when the temperature reaches a certain threshold, passing the temperature value along with the event:

```csharp
using System;

// Custom event arguments class inheriting from EventArgs
public class TemperatureEventArgs : EventArgs
{
    public double Temperature { get; }

    public TemperatureEventArgs(double temperature)
    {
        Temperature = temperature;
    }
}

// Publisher class with an event using custom event arguments
public class TemperatureSensor
{
    // Event declaration with custom event arguments
    public event EventHandler<TemperatureEventArgs> TemperatureThresholdReached;

    private double currentTemperature;

    public void SimulateTemperatureChange(double newTemperature)
    {
        currentTemperature = newTemperature;
        Console.WriteLine($"Current Temperature: {currentTemperature}");

        // Check if temperature threshold is reached (e.g., 100 degrees)
        if (currentTemperature >= 100)
        {
            OnTemperatureThresholdReached(new TemperatureEventArgs(currentTemperature));
        }
    }

    protected virtual void OnTemperatureThresholdReached(TemperatureEventArgs e)
    {
        TemperatureThresholdReached?.Invoke(this, e); // Raise the event
    }
}

// Subscriber class handling the event
public class TemperatureMonitor
{
    public TemperatureMonitor(TemperatureSensor sensor)
    {
        // Subscribe to the event
        sensor.TemperatureThresholdReached += Sensor_TemperatureThresholdReached;
    }

    // Event handler method
    private void Sensor_TemperatureThresholdReached(object sender, TemperatureEventArgs e)
    {
        Console.WriteLine($"Temperature threshold reached! Temperature: {e.Temperature}");
        // Take action based on the temperature information
    }
}

public class Program
{
    public static void Main()
    {
        TemperatureSensor sensor = new TemperatureSensor();
        TemperatureMonitor monitor = new TemperatureMonitor(sensor);

        // Simulate temperature changes
        sensor.SimulateTemperatureChange(95);
        sensor.SimulateTemperatureChange(105);
    }
}
```

### Explanation:

- **Custom Event Arguments Class (`TemperatureEventArgs`)**:
  - Inherits from `EventArgs`.
  - Contains a property `Temperature` to store the temperature value associated with the event.

- **TemperatureSensor Class**:
  - Declares an event `TemperatureThresholdReached` using `EventHandler<TemperatureEventArgs>`.
  - Raises the event `TemperatureThresholdReached` when the temperature threshold (100 degrees) is reached, passing a new instance of `TemperatureEventArgs` containing the current temperature.

- **TemperatureMonitor Class**:
  - Subscribes to the `TemperatureThresholdReached` event of `TemperatureSensor`.
  - Defines the event handler `Sensor_TemperatureThresholdReached` that processes the event and accesses the `Temperature` property from `TemperatureEventArgs`.

- **Program Class**:
  - Demonstrates the usage by creating instances of `TemperatureSensor` and `TemperatureMonitor`.
  - Simulates temperature changes and triggers the event when the threshold is exceeded.

### Benefits of Custom Event Arguments:

- **Encapsulation**: Custom event arguments encapsulate relevant data associated with the event, making the event handling logic more modular and understandable.
  
- **Flexibility**: Allows passing complex or additional information about the event that standard `EventArgs` may not provide.

- **Type Safety**: Helps in maintaining type safety by ensuring that event handler methods receive the correct types of arguments.

### Conclusion:

Custom event arguments in C# provide a structured and flexible way to pass additional information when raising events. By inheriting from `EventArgs`, you can encapsulate data specific to your application's needs, enhancing the clarity and extensibility of event-driven programming patterns. Understanding how to implement and use custom event arguments is essential for designing responsive and modular applications in C#.