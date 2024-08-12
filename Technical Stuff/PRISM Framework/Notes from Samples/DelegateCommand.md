## RaiseCanExecuteChange

^146beb

- View:
```xml
<CheckBox IsChecked="{Binding IsEnabled}" Content="Can Execute Command" Margin="10"/>
<Button Command="{Binding ExecuteDelegateCommand}" Content="DelegateCommand" Margin="10"/>
```
- ViewModel:
```csharp
public DelegateCommand ExecuteDelegateCommand { get; private set; }

private bool _isEnabled;
public bool IsEnabled
{
	get { return _isEnabled; }
	set
	{
		SetProperty(ref _isEnabled, value);
		SubmitCommand.RaiseCanExecuteChanged();
	}
}

public MainWindowViewModel()
{
	ExecuteDelegateCommand = new DelegateCommand(Execute, CanExecute);
}

private void Execute()
{
	UpdateText = $"Updated: {DateTime.Now}";
}

private bool CanExecute()
{
	return IsEnabled;
}
```
```ad-note
This is how the code goes: PropertyChanged -> RaiseCanExecuteChanged -> Control in UI which use Command handle -> CanExecute -> Update Control UI
```

## ObservesProperty

^d3f28c

- View:
```xml
<CheckBox IsChecked="{Binding IsEnabled}" Content="Can Execute Command" Margin="10"/>
<Button Command="{Binding DelegateCommandObservesProperty}" Content="DelegateCommand ObservesProperty" Margin="10"/>
```
- ViewModel:
```csharp
public DelegateCommand DelegateCommandObservesProperty { get; private set; }

private bool _isEnabled;
public bool IsEnabled
{
	get { return _isEnabled; }
	set
	{
		SetProperty(ref _isEnabled, value);
		SubmitCommand.RaiseCanExecuteChanged();
	}
}

public MainWindowViewModel()
{
	DelegateCommandObservesProperty = new DelegateCommand(Execute, CanExecute).ObservesProperty(() => IsEnabled);
}

private void Execute()
{
	UpdateText = $"Updated: {DateTime.Now}";
}

private bool CanExecute()
{
	return IsEnabled;
}
```
## ObservesCanExecute

^cd0217

- View:
```xml
<CheckBox IsChecked="{Binding IsEnabled}" Content="Can Execute Command" Margin="10"/>
<Button Command="{Binding DelegateCommandObservesCanExecute}" Content="DelegateCommand ObservesCanExecute" Margin="10"/>
```
- ViewModel:
```csharp
public DelegateCommand DelegateCommandObservesProperty { get; private set; }

private bool _isEnabled;
public bool IsEnabled
{
	get { return _isEnabled; }
	set
	{
		SetProperty(ref _isEnabled, value);
		SubmitCommand.RaiseCanExecuteChanged();
	}
}

public MainWindowViewModel()
{
	DelegateCommandObservesCanExecute = new DelegateCommand(Execute).ObservesCanExecute(() => IsEnabled);
}

private void Execute()
{
	UpdateText = $"Updated: {DateTime.Now}";
}

private bool CanExecute()
{
	return IsEnabled;
}
```
