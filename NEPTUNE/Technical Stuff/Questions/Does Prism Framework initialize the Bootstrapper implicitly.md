Yes, Prism does initialize the bootstrapper implicitly in a WPF (Windows Presentation Foundation) application. The Prism library is designed to follow the Bootstrapper pattern, where a bootstrapper is responsible for initializing and configuring the application, setting up the container, and preparing the shell.

Here's a brief overview of how the bootstrapper works in Prism:

1. **Bootstrapper Class:**
   - You typically create a custom bootstrapper class in your WPF application that inherits from `Prism.Bootstrapper`.
   - This class overrides various methods to configure and initialize the Prism infrastructure.

2. **Initialization Sequence:**
   - When your WPF application starts, Prism automatically discovers and initializes the bootstrapper.
   - The `OnStartup` method in your custom bootstrapper class is called automatically during the application startup process.

3. **Container Setup:**
   - In the `ConfigureContainer` method, you have the opportunity to configure the IoC (Inversion of Control) container. Prism uses an IoC container for dependency injection and service location.

4. **Module Loading:**
   - The `ConfigureModuleCatalog` method allows you to define and load modules. Modules in Prism are units of functionality that can be dynamically loaded and unloaded.

5. **Shell Creation:**
   - The `CreateShell` method is where you create the main application shell (main window). The shell is the container for the visual presentation of your application.

6. **Initialize Shell:**
   - The `InitializeShell` method is called after the shell is created. You can use this method to set the `MainWindow` property of the application and perform any additional shell initialization.

7. **Module Initialization:**
   - After the shell is created, Prism automatically initializes the modules defined in the `ConfigureModuleCatalog` method.

8. **Application Run:**
   - Finally, the WPF application's `Run` method is called, and the application starts its main message loop.

Here is a simplified example of a custom bootstrapper class:

```csharp
public class MyBootstrapper : Bootstrapper
{
    protected override DependencyObject CreateShell()
    {
        return Container.Resolve<MainWindow>();
    }

    protected override void InitializeShell()
    {
        Application.Current.MainWindow = (Window)Shell;
        Application.Current.MainWindow.Show();
    }
    
    // Other overrides for ConfigureContainer, ConfigureModuleCatalog, etc.
}
```

In this example, `MyBootstrapper` inherits from `Prism.Bootstrapper`, and the `CreateShell` and `InitializeShell` methods are overridden to create and initialize the main window of the application.

By following this pattern, Prism automates much of the initialization process, making it easier to set up a modular and maintainable WPF application.