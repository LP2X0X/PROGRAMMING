`IServiceLocator` is an interface in the Common Service Locator (CSL) library, which is a separate library designed to provide a common way to access services in .NET applications. The CSL library defines a set of interfaces that allow developers to use a consistent service location mechanism across different dependency injection containers.

`IServiceLocator` has methods for resolving services, checking if a service is available, and getting all instances of a service. While Prism historically used the Common Service Locator (CSL) pattern, it has moved away from it in recent versions, favoring more direct usage of dependency injection containers like Unity, Autofac, or others.

Here is an example of the `IServiceLocator` interface:

```csharp
public interface IServiceLocator
{
    object GetInstance(Type serviceType);
    object GetInstance(Type serviceType, string key);
    IEnumerable<object> GetAllInstances(Type serviceType);
    TService GetInstance<TService>();
    TService GetInstance<TService>(string key);
    IEnumerable<TService> GetAllInstances<TService>();
}
```

- `GetInstance`: Resolves a service of the specified type.
- `GetInstance` (with key): Resolves a named or keyed service.
- `GetAllInstances`: Resolves all instances of a service type.
- `GetInstance<TService>` and `GetAllInstances<TService>`: Generic overloads for type-safe resolution.

It's important to note that the use of Common Service Locator has been deprecated in Prism, and the framework encourages a more direct usage of the underlying dependency injection container (e.g., Unity, DryIoc, Autofac) through the `IContainerProvider` interface or equivalent container-specific APIs.

Here is an example using Prism's `IContainerProvider` for Unity:

```csharp
public class YourModule : IModule
{
    public void OnInitialized(IContainerProvider containerProvider)
    {
        IMyService myService = containerProvider.Resolve<IMyService>();
        // Use myService...
    }

    public void RegisterTypes(IContainerRegistry containerRegistry)
    {
        containerRegistry.Register<IMyService, MyServiceImpl>();
    }
}
```

In this example, `IContainerProvider` provides a more direct and type-safe way to access the container, eliminating the need for the Common Service Locator.