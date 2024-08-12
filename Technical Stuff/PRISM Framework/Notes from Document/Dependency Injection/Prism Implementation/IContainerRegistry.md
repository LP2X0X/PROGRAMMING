- `IContainerRegistry` is primarily used during the configuration phase of the application, typically in the `ConfigureContainer` method of the Prism `Bootstrapper`. It provides methods for registering types with the container.
- The `Register` method of `IContainerRegistry` is commonly used to register types with the container, specifying the interface or base type and the concrete type.
```csharp
public class YourModule : IModule
{
    public void RegisterTypes(IContainerRegistry containerRegistry)
    {
        containerRegistry.Register<IMyService, MyServiceImpl>();
    }

    public void OnInitialized(IContainerProvider containerProvider)
    {
        // Initialization logic
    }
}

```