## Definition
- Each module has a central class that is responsible for initializing the module and integrating its functionality into the application. That class implements the `IModule` interface.
- The `IModule` interface has two methods, named `OnInitialized` and `RegisterTypes`. Both take a reference to the dependency injection container as a parameter.
## Methods
- When a module is loaded into the application, `RegisterTypes` is called first and should be used to register any services or functionality that the module implements.
- Next the `OnInitialized` method is called. It is here that things like view registrations or any other module initialization code should be performed.
```csharp
public class MyModule : IModule
{
    public void RegisterTypes(IContainerRegistry containerRegistry)
    {
        // register with the container that SomeService implements ISomeService
        // ISomeService is defined in the Infrastructure module, see app architecture diagram
        containerRegistry.Register<MyApplication.Infrastructure.ISomeService, SomeService>();
    }

    public void OnInitialized(IContainerProvider containerProvider)
    {
        // use the containerProvider to retrieve the instance of the Prism RegionManager
        // and register the view in this module with a specific region in the app
        var regionManager = containerProvider.Resolve<IRegionManager>();
        regionManager.RegisterViewWithRegion("MyModuleView", typeof(Views.ThisModuleView));
    }
}
```