```csharp
public class SomeViewModel : BindableBase
{
    private IModuleManager _moduleManager = null;

    public SomeViewModel(IModuleManager moduleManager)
    {
        _moduleManager = moduleManager;
        _moduleManager.LoadModuleCompleted += _moduleManager_LoadModuleCompleted;
    }

    private void _moduleManager_LoadModuleCompleted(object sender, LoadModuleCompletedEventArgs e)
    {
        // ...
    }
}
```