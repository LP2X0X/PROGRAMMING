- Source code:
```csharp
using System;

namespace Prism
{
    /// <summary>
    /// Interface that defines if the object instance is active
    /// and notifies when the activity changes.
    /// </summary>
    public interface IActiveAware
    {
        /// <summary>
        /// Gets or sets a value indicating whether the object is active.
        /// </summary>
        /// <value><see langword="true" /> if the object is active; otherwise <see langword="false" />.</value>
        bool IsActive { get; set; }

        /// <summary>
        /// Notifies that the value for <see cref="IsActive"/> property has changed.
        /// </summary>
        event EventHandler IsActiveChanged;
    }
}

```
- By implementing the IActiveAware interface on your ViewModels, you will be notified when your view becomes active or inactive. When the view's active status changes, you can update the active status of the child commands. Then, when the user invokes the composite command, the command on the active child view will be invoked.
```csharp
public class TabViewModel : BindableBase, IActiveAware
{
	private bool _isActive;
	public bool IsActive
	{
		get { return _isActive; }
		set
		{
			_isActive = value;
			OnIsActiveChanged();
		}
	}

	public event EventHandler IsActiveChanged;

	public DelegateCommand UpdateCommand { get; private set; }

	public TabViewModel(IApplicationCommands applicationCommands)
	{
		UpdateCommand = new DelegateCommand(Update);
		applicationCommands.SaveCommand.RegisterCommand(UpdateCommand);
	}

	private void Update()
	{
		//implement logic
	}

	private void OnIsActiveChanged()
	{
		UpdateCommand.IsActive = IsActive; //set the command as active
		IsActiveChanged?.Invoke(this, new EventArgs()); //invoke the event for all listeners
	}
}
```