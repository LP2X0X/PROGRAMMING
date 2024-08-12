## Definition
- A bootstrapper is a class that is responsible for the initialization of an application built using the Prism Library. The Prism bootstrapping process includes creating and configuring a module catalog, creating a dependency injection container such as Unity, configuring default region adapter for UI composition, creating and initializing the shell view, and initializing modules.
![[Ch2BootstrapperFig1.png|center]]
```ad-note
The Prism Library provides some [[Difference between PrismBootstrapperBase and PrismApplicationBase|additional base classes]], derived from Bootstrapper, that have default implementations that are appropriate for most applications. The only stages left for your application bootstrapper to implement are creating and initializing the shell.
```
## Create a Bootstrapper
### Implement the CreateShell Method 
- The CreateShell method allows a developer to specify the top-level window for a Prism application. The shell is usually the MainWindow or MainPage. Implement this method by returning an instance of your application's shell class.
- An example of using the ServiceLocator to resolve the shell object is shown in the following code example:
```csharp
protected override DependencyObject CreateShell()
{
    return ServiceLocator.Current.GetInstance<Shell>();
}
```
### Implement the InitializeShell Method
- After you create a shell, you may need to run initialization steps to ensure that the shell is ready to be displayed. For WPF applications, you will create the shell application object and set it as the application's main window:
```csharp
protected override void InitializeShell()
{
    Application.Current.MainWindow = Shell;
    Application.Current.MainWindow.Show();
}
```
```ad-note
The base implementation of InitializeShell does nothing. It is safe to not call the base class implementation.
```
