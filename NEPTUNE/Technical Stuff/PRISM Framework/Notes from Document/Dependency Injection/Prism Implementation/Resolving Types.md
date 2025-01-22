- In general, when a type is resolved, one of three things happens:
    - If the type has not been registered, the container throws an exception.
	    - Note: Some containers, including Unity, allow you to resolve a concrete type that has not been registered.
    - If the type has been registered as a singleton, the container returns the singleton instance. If this is the first time the type was called for, the container creates it and holds on to it for future calls.
    - If the type has not been registered as a singleton, the container returns a new instance.
	    - Note: By default, types registered with MEF are singletons and the container holds a reference to the object. In Unity, new instances of objects are returned by default, and the container does not maintain a reference to the object.
- Here are some common ways to resolve instances:
#### 1 - Implicitly
- When a class (e.g., a view model) has dependencies, the DI container resolves and injects them automatically based on the constructor parameters.
```csharp
public class YourViewModel : BindableBase
{
    private readonly IMyService _myService;

    public YourViewModel(IMyService myService)
    {
        _myService = myService;
    }

    // Use _myService in your view model logic
}
```
#### 2 - Explicitly
```csharp
public class YourViewModel : BindableBase
{
    private readonly IMyService _myService;

    public YourViewModel(IContainerProvider containerProvider)
    {
        _myService = containerProvider.Resolve<IMyService>();
    }

    // Use _myService in your view model logic
}
```
#### 3 - By name
- When registering:
```csharp
containerRegistry.Register<IDBService, DaoDBService>("DAO");
containerRegistry.Register<IDBService, SqlServerDBService>("SQLSERVER");
```
- When resolving:
```csharp
IMyService daoService = container.Resolve<IMyService>("DAO");
IMyService sqlServerService = container.Resolve<IMyService>("SQLSERVER");
```
#### 4 - All Implementations
- If multiple implementations of an interface are registered, you can use ResolveAll to get all registered implementations:
```csharp
IEnumerable<IMyService> allServices = container.ResolveAll<IMyService>();
```