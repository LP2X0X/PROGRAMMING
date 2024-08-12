- Prism provides the DelegateCommand implementation of the [[ICommand]] interface that you can readily use in your applications.
## Create a DelegateCommand
- The Prism DelegateCommand class implements the ICommand interface's Execute and CanExecute methods by invoking these delegates.
- You specify the delegates to your ViewModel methods in the DelegateCommand class constructor.
```ad-example
````csharp
public class ArticleViewModel
{
    public DelegateCommand SubmitCommand { get; private set; }

    public ArticleViewModel()
    {
        SubmitCommand = new DelegateCommand<object>(Submit, CanSubmit);
    }

    void Submit(object parameter)
    {
        //implement logic
    }

    bool CanSubmit(object parameter)
    {
        return true;
    }
}
```
- Related source code:
```csharp
        public DelegateCommand(Action executeMethod, Func<bool> canExecuteMethod)
            : base()
        {
            if (executeMethod == null || canExecuteMethod == null)
                throw new ArgumentNullException(nameof(executeMethod), Resources.DelegateCommandDelegatesCannotBeNull);

            _executeMethod = executeMethod;
            _canExecuteMethod = canExecuteMethod;
        }

```
```ad-note
When the Execute method is called on the DelegateCommand object, it simply forwards the call to the method in your ViewModel class via the delegate that you specified in the constructor. The DelegateCommand class is a generic type. The type argument specifies the type of the command parameter passed to the Execute and CanExecute methods. In the next example, the command parameter is of type object.
````csharp
/// <summary>
/// Handle the internal invocation of <see cref="ICommand.Execute(object)"/>
/// </summary>
/// <param name="parameter">Command Parameter</param>
protected override void Execute(object? parameter)
{
	Execute();
}

///<summary>
/// Executes the command.
///</summary>
public void Execute()
{
	try
	{
		_executeMethod();
	}
	catch (Exception ex)
	{
		if (!ExceptionHandler.CanHandle(ex))
			throw;

		ExceptionHandler.Handle(ex, null);
	}
}

/// <summary>
/// Handle the internal invocation of <see cref="ICommand.CanExecute(object)"/>
/// </summary>
/// <param name="parameter"></param>
/// <returns><see langword="true"/> if the Command Can Execute, otherwise <see langword="false" /></returns>
protected override bool CanExecute(object? parameter)
{
	return CanExecute();
}

/// <summary>
/// Determines if the command can be executed.
/// </summary>
/// <returns>Returns <see langword="true"/> if the command can execute,otherwise returns <see langword="false"/>.</returns>
public bool CanExecute()
{
	try
	{
		return _canExecuteMethod();
	}
	catch (Exception ex)
	{
		if (!ExceptionHandler.CanHandle(ex))
			throw;

		ExceptionHandler.Handle(ex, null);

		return false;
	}
}
```

```ad-note
The DelegateCommand class is a generic type. The type argument specifies the type of the command parameter passed to the Execute and CanExecute methods. In the preceding example, the command parameter is of type object.
```
## Invoking DelegateCommands from the View
- There are a number of ways in which a control in the view can be associated with a command object provided by the ViewModel. Certain WPF, Xamarin.Forms, and UWP controls can be easily data bound to a command object through the Command property.
```xml
<Button Command="{Binding SubmitCommand}" CommandParameter="OrderId"/>
```
- A command parameter can also be optionally defined using the `CommandParameter` property. The type of the expected argument is specified in the `DelegateCommand<T>` generic declaration. The control will automatically invoke the target command when the user interacts with that control, and the command parameter, if provided, will be passed as the argument to the command's `Execute` method. In the preceding example, the button will automatically invoke the `SubmitCommand` when it is clicked. Additionally, if a `CanExecute` delegate is specified, the button will be automatically disabled if `CanExecute` returns `false`, and it will be enabled if it returns `true`.
## Raising Change Notification
- The ViewModel often needs to indicate a change in the command's CanExecute status so that any controls in the UI that are bound to the command will update their enabled status to reflect the availability of the bound command. The DelegateCommand provides several ways to send these notifications to the UI.
### RaiseCanExecuteChanged
- Use the [[Technical Stuff/PRISM Framework/Notes from Samples/DelegateCommand#^146beb|RaiseCanExecuteChanged]] method whenever you need to manually update the state of the bound UI elements. For example, when the IsEnabled property values changes, we are calling RaiseCanExecuteChanged in the setter of the property to notify the UI of state changes.
### ObservesProperty
- In cases where the command should send notifications when a property value changes, you can use the [[Technical Stuff/PRISM Framework/Notes from Samples/DelegateCommand#^d3f28c|ObservesProperty]] method. When using the ObservesProperty method, whenever the value of the supplied property changes, the DelegateCommand will automatically call RaiseCanExecuteChanged to notify the UI of state changes.
```ad-important
You can chain-register multiple properties for observation when using the ObservesProperty method. Example:
````csharp
ObservesProperty(() => IsEnabled).ObservesProperty(() => CanSave).
```
### ObservesCanExecute
- If your CanExecute is the result of a simple Boolean property, you can eliminate the need to declare a CanExecute delegate, and use the [[Technical Stuff/PRISM Framework/Notes from Samples/DelegateCommand#^cd0217|ObservesCanExecute]] method instead. ObservesCanExecute will not only send notifications to the UI when the registered property value changes but it will also **use that same property as the actual CanExecute delegate**.
```ad-warning
Do not attempt to chain-register ObservesCanExecute methods. Only one property can be observed for the CanExcute delegate.
```

### Implementing a Task-Based DelegateCommand
- In today's world of async/await, calling asynchronous methods inside of the Execute delegate is a very common requirement. Everyone's first instinct is that they need an AsyncCommand, but that assumption is wrong. ICommand by nature is synchronous, and the Execute and CanExecute delegates should be considered events. This means that async void is a perfectly valid syntax to use for commands. There are two approaches to using async methods with DelegateCommand.
- Option 1:
```csharp
public class ArticleViewModel
{
    public DelegateCommand SubmitCommand { get; private set; }

    public ArticleViewModel()
    {
        SubmitCommand = new DelegateCommand(Submit);
    }

    async void Submit()
    {
        await SomeAsyncMethod();
    }
}
```
- Option 2:
```csharp
public class ArticleViewModel
{
    public DelegateCommand SubmitCommand { get; private set; }

    public ArticleViewModel()
    {
        SubmitCommand = new DelegateCommand(async ()=> await Submit());
    }

    Task Submit()
    {
        return SomeAsyncMethod();
    }
}
```