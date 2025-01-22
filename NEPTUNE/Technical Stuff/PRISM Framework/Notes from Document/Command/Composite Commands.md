- The CompositeCommand class represents a command that is composed from multiple child commands. When the composite command is invoked, each of its child commands is invoked in turn. It is useful in situations where you need to represent a group of commands as a single command in the UI or where you want to invoke multiple commands to implement a logical command.
- The CompositeCommand class maintains a list of child commands (DelegateCommand instances). The Execute method of the CompositeCommand class simply calls the Execute method on each of the child commands in turn. The CanExecute method similarly calls the CanExecute method of each child command, but if any of the child commands cannot be executed, the CanExecute method will return false. In other words, by default, a CompositeCommand can only be executed when all the child commands can be executed.
## Create Composite Command
- To create a composite command, instantiate a CompositeCommand instance and then expose it as either an ICommand or ComponsiteCommand property.
```csharp
public class ApplicationCommands
{
	private CompositeCommand _saveCommand = new CompositeCommand();
	public CompositeCommand SaveCommand
	{
		get { return _saveCommand; }
	}
}
```
## Make it globally available
- It's important that when you register a child command with a CompositeCommand that you are using the same instance of the CompositeCommand throughout the application. This requires the CompositeCommand to be defined as a singleton in your application. This can be done by either using dependency injection (DI), or by defining your CompositeCommand as a static class.
### Using Dependency Injection
- Create an interface:
```csharp
public interface IApplicationCommands
{
	CompositeCommand SaveCommand { get; }
}
```
- Next, create a class that implements the interface:
```csharp
public class ApplicationCommands : IApplicationCommands
{
	private CompositeCommand _saveCommand = new CompositeCommand();
	public CompositeCommand SaveCommand
	{
		get { return _saveCommand; }
	}
}
```
- Once you have defined your ApplicationCommands class, you must register it as a singleton with the container:
```csharp
public partial class App : PrismApplication
{
	protected override void RegisterTypes(IContainerRegistry containerRegistry)
	{
		containerRegistry.RegisterSingleton<IApplicationCommands, ApplicationCommands>();
	}
}
```
- Next, ask for the IApplicationCommands interface in the ViewModel constructor. Once you have an instance of the ApplicationCommands class, can now register your DelegateCommands with the appropriate CompositeCommand.
```csharp
public DelegateCommand UpdateCommand { get; private set; }

public TabViewModel(IApplicationCommands applicationCommands)
{
	UpdateCommand = new DelegateCommand(Update);
	applicationCommands.SaveCommand.RegisterCommand(UpdateCommand);
}
```
## Binding to composite command
### Using Dependency Injection
- When using DI, you must expose the IApplicationCommands for binding to a View. In the ViewModel of the view, ask for the IApplicationCommands in the constructor and set a property of type IApplicationCommands to the instance.
```csharp
public class MainWindowViewModel : BindableBase
{
	private IApplicationCommands _applicationCommands;
	public IApplicationCommands ApplicationCommands
	{
		get { return _applicationCommands; }
		set { SetProperty(ref _applicationCommands, value); }
	}

	public MainWindowViewModel(IApplicationCommands applicationCommands)
	{
		ApplicationCommands = applicationCommands;
	}
}
```
- In the view, bind the button to the ApplicationCommands.SaveCommand property. The SaveCommand is a property that is defined on the ApplicationCommands class.
```xml
<Button Content="Save" Command="{x:Static local:ApplicationCommands.SaveCommand}" />
```
## Unregister a Command
- As seen in the previous examples, child commands are registered using the CompositeCommand.RegisterCommand method. However, when you no longer wish to respond to a CompositeCommand or if you are destroying the View/ViewModel for garbage collection, you should unregister the child commands with the CompositeCommand.UnregisterCommand method.
```csharp
public void Destroy()
{
	_applicationCommands.UnregisterCommand(UpdateCommand);
}
```
```ad-important
You MUST unregister your commands from a CompositeCommand when the View/ViewModel is no longer needed (ready for GC). Otherwise you will have introduced a memory leak.
```
## Execute commands on active view
- Composite commands at the parent view level will often be used to coordinate how commands at the child view level are invoked. In some cases, you will want the commands for all shown views to be executed, as in the Save All command example described earlier. In other cases, you will want the command to be executed only on the active view. In this case, the composite command will execute the child commands only on views that are deemed to be active; it will not execute the child commands on views that are not active. For example, you may want to implement a Zoom command on the application's toolbar that causes only the currently active item to be zoomed, as shown in the following diagram.
- To support this scenario, Prism provides the IActiveAware interface. The IActiveAware interface defines an IsActive property that returns true when the implementer is active, and an IsActiveChanged event that is raised whenever the active state is changed.
- When the monitorCommandActivity parameter is true, the CompositeCommand class exhibits the following behavior:
    - CanExecute: Returns true only when all active commands can be executed. Child commands that are inactive will not be considered at all.
    - Execute: Executes all active commands. Child commands that are inactive will not be considered at all.
```csharp
public class ApplicationCommands : IApplicationCommands
{
	private CompositeCommand _saveCommand = new CompositeCommand(true);
	public CompositeCommand SaveCommand
	{
		get { return _saveCommand; }
	}
}
```