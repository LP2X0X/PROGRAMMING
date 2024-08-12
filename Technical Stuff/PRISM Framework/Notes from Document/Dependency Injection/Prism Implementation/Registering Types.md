## 0 - Check if a Service has been Registered
```ad-important
You should only use IsRegistered to check for a dependency if your intent is to register a default implementation.
```
```csharp
if (containerRegistry.IsRegistered<ISomeService>())
{
    // Do something...
}
```

## 1 - Transient Services
- For those services that you expect to create a new instance each time it is created you will simply call the `Register` method and provide the Service Type and the Implementing Type, except in cases where it may be appropriate to simply register the concrete type.
```csharp
// Where it will be appropriate to use FooService as a concrete type
containerRegistry.Register<FooService>(); // FooService might be a type (class) here

containerRegistry.Register<IBarService, BarService>();
```

## 2 - Singleton Services
- Many times you may have a service which is used throughout your application. As a result it would not be a good idea to create a new instance every time you need the service. In order to provide better memory management it is therefore a better practice to make such services a Singleton that can be used throughout the application.
- There are also many times in which you may need a service that retains it state throughout the lifecycle of your application. For either of these cases it makes far more sense to register your service as a Singleton.
```csharp
// Where it will be appropriate to use FooService as a concrete type
containerRegistry.RegisterSingleton<FooService>();

containerRegistry.RegisterSingleton<IBarService, BarService>();
```
## 3 - Service Instance
- Registering a service instance in a dependency injection (DI) container involves informing the container about a specific instance of a type that should be used when that type is requested. This is useful when you want to control the creation of an object and provide a pre-existing instance rather than allowing the container to create it.
```csharp
containerRegistry.RegisterInstance<IFoo>(new FooImplementation());

// Sample of using James Montemagno's Monkey Cache
Barrel.ApplicationId = "your_unique_name_here";
containerRegistry.RegisterInstance<IBarrel>(Barrel.Current);
```
## Lazy Resolution
- In Prism, lazy resolution refers to the concept of deferring the creation of an object until it is actually requested or needed. This is achieved using the `Lazy<T>` class in .NET, and it can be particularly useful in scenarios where the creation of an object might be resource-intensive or when you want to delay the initialization of a service until it is explicitly required.
```csharp
using Prism.Ioc;
using System;

public interface IExpensiveService
{
    void PerformExpensiveOperation();
}

public class ExpensiveService : IExpensiveService
{
    public ExpensiveService()
    {
        Console.WriteLine("ExpensiveService is being constructed.");
    }

    public void PerformExpensiveOperation()
    {
        Console.WriteLine("Expensive operation performed.");
    }
}

public class SomeViewModel
{
    private readonly Lazy<IExpensiveService> lazyExpensiveService;

    public SomeViewModel(Lazy<IExpensiveService> lazyExpensiveService)
    {
        this.lazyExpensiveService = lazyExpensiveService;
    }

    public void DoSomething()
    {
        // The ExpensiveService instance is created only when needed
        IExpensiveService expensiveService = lazyExpensiveService.Value;
        expensiveService.PerformExpensiveOperation();
    }
}

class Program
{
    static void Main()
    {
        var container = new ContainerProvider().CreateContainer();
        
        // Register IExpensiveService with Lazy resolution
        container.Register<IExpensiveService, ExpensiveService>(IfAlreadyRegistered.Replace);

        // Resolve SomeViewModel
        var viewModel = container.Resolve<SomeViewModel>();

        // Do something triggering the lazy resolution
        viewModel.DoSomething();
    }
}
```
## Resolve All
- In Prism, the ResolveAll method is used to retrieve all instances of a specified type from the container. This is particularly useful when you have multiple implementations of an interface or an abstract class registered with the dependency injection container, and you want to obtain a collection of all the registered instances.
```csharp
using Prism.Ioc;

public interface IService
{
    void DoSomething();
}

public class ServiceA : IService
{
    public void DoSomething()
    {
        Console.WriteLine("ServiceA is doing something.");
    }
}

public class ServiceB : IService
{
    public void DoSomething()
    {
        Console.WriteLine("ServiceB is doing something.");
    }
}

class Program
{
    static void Main()
    {
        var container = new ContainerProvider().CreateContainer();

        // Register multiple implementations of IService
        container.Register<IService, ServiceA>();
        container.Register<IService, ServiceB>();

        // Resolve all instances of IService
        var allServices = container.ResolveAll<IService>();

        // Iterate and use each instance
        foreach (var service in allServices)
        {
            service.DoSomething();
        }
    }
}
```

---

## Register view 
- By default, **RegisterForNavigation** will use the **Name** of the Page type as the unique identifier/key. The following code snippet results in a mapping between the MainPage type, and the unique identifier/key of "MainPage". This means when you request to navigate to the MainPage, you will provide the string "MainPage" as the navigation target.

```csharp
protected override void RegisterTypes(IContainerRegistry containerRegistry)
{
    containerRegistry.RegisterForNavigation<MainPage>();
}
```
- To navigate to the MainPage using this registration method:
```csharp
_navigationService.NavigateAsync("MainPage");
```
- By default, Prism uses a convention to find the ViewModel for the view (PageName + ViewModel = PageNameViewModel). This process uses reflection which introduces a small performance hit. If you would like to avoid this performance hit, you can provide the ViewModel type when registering your Page for navigation.
```csharp
protected override void RegisterTypes(IContainerRegistry containerRegistry)
{
    containerRegistry.RegisterForNavigation<MainPage, MainPageViewModel>();
}
```
