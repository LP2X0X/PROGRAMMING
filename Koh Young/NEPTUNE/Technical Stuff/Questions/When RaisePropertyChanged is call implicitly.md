In the context of WPF and the MVVM (Model-View-ViewModel) pattern, the `RaisePropertyChanged` method is typically used to notify the binding system that a property has changed, and the UI needs to be updated accordingly.

When you set a property to a completely new object, the property change is detected, and the `RaisePropertyChanged` method is called automatically. This is because the binding system relies on the `INotifyPropertyChanged` interface, and it uses the notification to update the UI.

However, if you modify the existing object without replacing it with a new one, the property change may not be detected automatically, and you might need to manually call `RaisePropertyChanged` to ensure that the UI is updated.

For example, consider a class with a property like this:

```csharp
private string _myProperty;

public string MyProperty
{
    get { return _myProperty; }
    set
    {
        if (_myProperty != value)
        {
            _myProperty = value;
            RaisePropertyChanged(nameof(MyProperty));
        }
    }
}
```

In this example, the `RaisePropertyChanged` method is explicitly called only when the value of `MyProperty` is actually changed. If you modify the existing string without changing the reference (e.g., updating its content), you would need to manually call `RaisePropertyChanged` to ensure the UI is updated.

```csharp
MyProperty = MyProperty + "Updated"; // This will trigger RaisePropertyChanged
```

It's important to note that this behavior is specific to the implementation of the `RaisePropertyChanged` method and how your properties are implemented in your ViewModel. If you're using a base class or a tool like Fody.PropertyChanged, the behavior might be different. Always refer to the documentation or source code of the specific implementation you're using for accurate information.