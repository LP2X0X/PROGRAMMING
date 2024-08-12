- In the PubSubEvent class, the Subscribe method's parameter is Action type => The method which handle the event **must return void**. ^d998b3
```csharp
public SubscriptionToken Subscribe(Action action)
{
	return Subscribe(action, ThreadOption.PublisherThread);
}
```
---
- The Subscribe method also had many other overload methods. ^b57119
```csharp
public SubscriptionToken Subscribe(Action action, ThreadOption threadOption)
public SubscriptionToken Subscribe(Action action, bool keepSubscriberReferenceAlive)
public virtual SubscriptionToken Subscribe(Action action, ThreadOption threadOption, bool keepSubscriberReferenceAlive)
public virtual SubscriptionToken Subscribe(Action<TPayload> action, ThreadOption threadOption, bool keepSubscriberReferenceAlive, Predicate<TPayload> filter)
etc...
```
---