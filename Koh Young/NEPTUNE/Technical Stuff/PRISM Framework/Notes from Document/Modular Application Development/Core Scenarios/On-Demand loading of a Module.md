- After a module is specified as on-demand, the application can ask the module to be loaded. The code that wants to initiate the loading needs to obtain a reference to the `IModuleManager` service registered with the container in the `App` class.
```csharp
public class SomeViewModel : BindableBase
{
    private IModuleManager _moduleManager = null;

    public SomeViewModel(IModuleManager moduleManager)
    {
        // use dependency injection to get the module manager
        _moduleManager = moduleManager;
    }

    private void LoadSomeModule(string moduleName)
    {
        _moduleManager.LoadModule(moduleName);
    }
}
```