Each module has a central class that is responsible for initializing the module and integrating its functionality into the application. That class implements the `IModule` interface, as shown here.
```csharp
public class MyModule : IModule
{
    public void RegisterTypes(IContainerRegistry containerRegistry)
    {
    }

    public void OnInitialized(IContainerProvider containerProvider)
    {
    }
}
```
- Implement `RegisterTypes` to handle the registration with the dependency injection container all of the **services that this module implements**.
- How `OnInitialized` method is implemented will depend on the requirements of your application. Here is where you can **register** your **views** and do any other module level initialize that may be required.