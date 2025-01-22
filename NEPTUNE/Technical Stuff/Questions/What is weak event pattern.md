https://learn.microsoft.com/en-us/dotnet/desktop/wpf/events/weak-event-patterns?view=netdesktop-8.0
The weak event pattern is a design pattern used to prevent memory leaks in scenarios where an event source holds references to event subscribers (event handlers) for an extended period. It ensures that event handlers can be garbage collected even if they are still registered with the event source.

In the weak event pattern, weak references are used to hold references to the event handlers. A `WeakReference` is a special type of reference in .NET that allows the garbage collector to collect the referenced object if there are no other strong references to it.

Here's a simplified explanation of how the weak event pattern works and how it allows event handlers to be garbage collected:

1. **Strong and Weak References:**
   - In a standard event registration, the event source holds a strong reference to the event handler. As long as the event source exists, the event handler cannot be garbage collected because there is still a strong reference to it.

   - In the weak event pattern, a `WeakReference` is used to hold a weak reference to the event handler. This allows the event handler to be garbage collected if there are no other strong references to it.

2. **WeakEventManager (Optional):**
   - In some implementations of the weak event pattern, a `WeakEventManager` class is used to manage weak event subscriptions. This class can be responsible for handling the weak references and dispatching events.

3. **Garbage Collection and Finalization:**
   - When the garbage collector runs, it identifies objects that are no longer reachable. If an object is only referenced by weak references, it is considered eligible for garbage collection.

4. **EventHandler Cleanup:**
   - If an event handler is no longer reachable by strong references (i.e., the event source is the only reference), the `WeakReference` to the event handler becomes eligible for garbage collection.

5. **Automatic Unsubscription:**
   - As the weak references are collected, the weak event pattern ensures that the corresponding event subscriptions are automatically unsubscribed. This prevents the event source from attempting to invoke event handlers that have been garbage collected.

6. **Potential WeakEventManager Usage (Optional):**
   - In some implementations, the `WeakEventManager` might use a combination of weak references and event registration to manage the weak subscriptions more efficiently.

Here's a basic example in C#:

```csharp
public class EventSource
{
    private readonly WeakEvent<EventHandler> weakEvent = new WeakEvent<EventHandler>();

    public event EventHandler MyEvent
    {
        add => weakEvent.Add(value);
        remove => weakEvent.Remove(value);
    }

    public void RaiseEvent()
    {
        weakEvent.Invoke(this, EventArgs.Empty);
    }
}

public class WeakEvent<TEventHandler> where TEventHandler : class
{
    private readonly List<WeakReference> eventHandlers = new List<WeakReference>();

    public void Add(TEventHandler handler)
    {
        eventHandlers.Add(new WeakReference(handler));
    }

    public void Remove(TEventHandler handler)
    {
        eventHandlers.RemoveAll(reference => !reference.IsAlive || reference.Target == handler);
    }

    public void Invoke(object sender, EventArgs e)
    {
        foreach (var weakReference in eventHandlers)
        {
            if (weakReference.IsAlive)
            {
                var handler = weakReference.Target as TEventHandler;
                handler?.Invoke(sender, e);
            }
        }
    }
}
```

This is a simplified example to illustrate the basic concept. In real-world scenarios, you might encounter more sophisticated implementations, especially when using frameworks or libraries that provide built-in support for weak events.