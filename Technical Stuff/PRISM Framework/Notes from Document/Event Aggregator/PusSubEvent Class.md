- The real work of connecting publishers and subscribers is done by the PubSubEvent class. This is the only implementation of the EventBase class that is included in the Prism Library. This class maintains the list of subscribers and handles event dispatching to the subscribers.
## Create an event
- The `PubSubEvent<TPayload>` is intended to be the base class for an application's or module's specific events.
- `TPayLoad` is the type of the event's payload. The **payload** is the argument that will be **passed to subscribers** when the event is published.
```csharp
public class TickerSymbolSelectedEvent : PubSubEvent<string>{}
```
```ad-note
In a composite application, the events are frequently shared between multiple modules, so they are defined in a common place. It is common practice to define these events in a shared assembly such as a "Core" or "Infrastructure" project.
```
## Publish an Event
- **Publishers raise an event** by **retrieving** the event from the `EventAggregator` and calling the `Publish` method.
```csharp
_eventAggregator.GetEvent<TickerSymbolSelectedEvent>().Publish("STOCK0");
```
- To access the `EventAggregator`, you can use dependency injection by adding a parameter of type `IEventAggregator` to the class constructor.
```csharp
// Inject EventAggregator
public class MainPageViewModel
{
    IEventAggregator _eventAggregator;
    public MainPageViewModel(IEventAggregator ea)
    {
        _eventAggregator = ea;
    }
}
```

## Subscribing to Event
- Subscribers can enlist with an event using one of the `Subscribe` method overloads available on the `PubSubEvent` class (More in [[EventAggregator#^b57119|note]]).
```csharp
public class MainPageViewModel
{
    public MainPageViewModel(IEventAggregator ea)
    {
        ea.GetEvent<TickerSymbolSelectedEvent>().Subscribe(ShowNews);
    }

    void ShowNews(string companySymbol) // This method return void for a reason
    {
        //implement logic
    }
}
```
### Subscribing on the UI thread
- Frequently, subscribers will need to update UI elements in response to events. In WPF, only a UI thread can update UI elements.
- By default, the subscriber receives the event on the publisher's thread. If the publisher sends the event from the UI thread, the subscriber can update the UI. However, if the publisher's thread is a background thread, the subscriber may be unable to directly update UI elements. In this case, the subscriber would need to schedule the updates on the UI thread using the [Dispatcher](https://learn.microsoft.com/en-us/dotnet/api/system.windows.threading.dispatcher?view=windowsdesktop-8.0) class.
- The `PubSubEvent` provided with the Prism Library can assist by allowing the subscriber to automatically receive the event on the UI thread.
```csharp
public class MainPageViewModel
{
    public MainPageViewModel(IEventAggregator ea)
    {
        ea.GetEvent<TickerSymbolSelectedEvent>().Subscribe(ShowNews, ThreadOption.UIThread);
    }

    void ShowNews(string companySymbol)
    {
        //implement logic
    }
}
```
- The following options are available for ThreadOption:
    - PublisherThread: Use this setting to receive the event on the publishers' thread. This is the default setting.
    - BackgroundThread: Use this setting to asynchronously receive the event on a .NET Framework thread-pool thread.
    - UIThread: Use this setting to receive the event on the UI thread.
```ad-note
In order for PubSubEvent to publish to subscribers on the UI thread, the EventAggregator must initially be constructed on the UI thread.
```
### Subscription Filtering
- *Subscribers may not need to handle every instance of a published event.* In these cases, the subscriber can use the filter parameter. The filter parameter is of type `System.Predicate<TPayLoad>` and is a delegate that gets executed when the event is published to determine if the payload of the published event matches a set of criteria required to have the subscriber callback invoked. If the payload does not meet the specified criteria, the subscriber callback is not executed.
```csharp
public class MainPageViewModel
{
    public MainPageViewModel(IEventAggregator ea)
    {
        TickerSymbolSelectedEvent tickerEvent = ea.GetEvent<TickerSymbolSelectedEvent>();
        tickerEvent.Subscribe(ShowNews, ThreadOption.UIThread, false, 
                              companySymbol => companySymbol == "STOCK0");
    }

    void ShowNews(string companySymbol)
    {
        //implement logic
    }
}
```
```ad-note
The Subscribe method returns a **subscription token** of type Prism.Events.SubscriptionToken that can be used to remove a subscription to the event later. This token is particularly useful when you are using anonymous delegates or lambda expressions as the callback delegate or when you are subscribing the same event handler with different filters.
```
```ad-warning
It is not recommended to modify the payload object from within a callback delegate because several threads could be accessing the payload object simultaneously. You could have the payload be immutable to avoid concurrency errors.
```
### Subscribing Using Strong References
- If you are raising multiple events in a short period of time and have noticed performance concerns with them, you may need to subscribe with strong delegate references. If you do that, you will then need to manually unsubscribe from the event when disposing the subscriber.
- By default, PubSubEvent maintains a weak delegate reference to the subscriber's handler and filter on subscription. This means the reference that PubSubEvent holds on to will not prevent garbage collection of the subscriber. Using a weak delegate reference relieves the subscriber from the need to unsubscribe and allows for proper garbage collection.
- However, maintaining this weak delegate reference is slower than a corresponding strong reference. For most applications, this performance will not be noticeable, but if your application publishes a large number of events in a short period of time, you may need to use strong references with PubSubEvent. If you do use strong delegate references, your subscriber should unsubscribe to enable proper garbage collection of your subscribing object when it is no longer used.
- To subscribe with a strong reference, use the keepSubscriberReferenceAlive parameter on the Subscribe method:
```csharp
public class MainPageViewModel
{
    public MainPageViewModel(IEventAggregator ea)
    {
        bool keepSubscriberReferenceAlive = true;
        TickerSymbolSelectedEvent tickerEvent = ea.GetEvent<TickerSymbolSelectedEvent>();
        tickerEvent.Subscribe(ShowNews, ThreadOption.UIThread, keepSubscriberReferenceAlive, 
                              companySymbol => companySymbol == "STOCK0");
    }

    void ShowNews(string companySymbol)
    {
        //implement logic
    }
}
```
### Unsubscribing from an Event
- If your subscriber no longer wants to receive events, you can unsubscribe by using your subscriber's handler or you can **unsubscribe** by using a **subscription token**.
- Directly unsubscribe:
```csharp
public class MainPageViewModel
{
    TickerSymbolSelectedEvent _event;
    public MainPageViewModel(IEventAggregator ea)
    {
        _event = ea.GetEvent<TickerSymbolSelectedEvent>();
        _event.Subscribe(ShowNews);
    }

    void Unsubscribe()
    {
        _event.Unsubscribe(ShowNews);
    }

    void ShowNews(string companySymbol)
    {
        //implement logic
    }
}
```
- Using subscription token:
```csharp
public class MainPageViewModel
{
    TickerSymbolSelectedEvent _event;
    SubscriptionToken _token;
    public MainPageViewModel(IEventAggregator ea)
    {
        _event = ea.GetEvent<TickerSymbolSelectedEvent>();
        _token = _event.Subscribe(ShowNews);
    }

    void Unsubscribe()
    {
        _event.Unsubscribe(_token);
    }

    void ShowNews(string companySymbol)
    {
        //implement logic
    }
}
```