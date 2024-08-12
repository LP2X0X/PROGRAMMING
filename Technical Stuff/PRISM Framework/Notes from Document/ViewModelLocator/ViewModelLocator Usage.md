- The Prism `ViewModelLocator` has an `AutoWireViewModel` attached property, that when set to `true` calls the `AutoWireViewModelChanged` method in the [[Technical Stuff/PRISM Framework/Notes from Document/ViewModelLocator/ViewModelLocationProvider|ViewModelLocationProvider]] class to resolve the ViewModel for the view, and then set the view’s data context to an instance of that ViewModel.
- To locate a ViewModel, the ViewModelLocationProvider **first attempts to resolve** the ViewModel from any mappings that may have been registered by the ViewModelLocationProvider.Register method (See Custom ViewModel Registrations). If the ViewModel cannot be resolved using this approach, the ViewModelLocationProvider **falls back to a convention-based ** approach to resolve the correct ViewModel type. 
- This convention assumes:
    - that ViewModels are in the same [[Technical Stuff/PRISM Framework/Notes from Samples/ViewModelLocationProvider#^e61d6d|assembly]] as the view types
    - that ViewModels are in a .ViewModels child [[Technical Stuff/PRISM Framework/Notes from Samples/ViewModelLocationProvider#^ae7e4c|namespace]]
    - that views are in a .Views child [[Technical Stuff/PRISM Framework/Notes from Samples/ViewModelLocationProvider#^1c0bb3|namespace]]
    - that ViewModel names correspond with view names and end with "ViewModel."

- Use the `AutoWireViewModel` attached property as below. Set the value to `False` to opt-out and `True` to explicitly opt-in.
```csharp
<Window x:Class="Demo.Views.MainWindow"
    ...
    xmlns:prism="http://prismlibrary.com/"
    prism:ViewModelLocator.AutoWireViewModel="False">
```
- Source code:
```csharp
/// <summary>
/// Automatically looks up the viewmodel that corresponds to the current view, using two strategies:
/// It first looks to see if there is a mapping registered for that view, if not it will fallback to the convention based approach.
/// </summary>
/// <param name="view">The dependency object, typically a view.</param>
/// <param name="setDataContextCallback">The call back to use to create the binding between the View and ViewModel</param>
public static void AutoWireViewModelChanged(object view, Action<object, object> setDataContextCallback)
{
	// Try mappings first
	object viewModel = GetViewModelForView(view); // Check for any custom viewmodel registration

	// try to use ViewModel type
	if (viewModel == null)
	{
		// check type mappings
		var viewModelType = GetViewModelTypeForView(view.GetType());

		// check platform View to ViewModel resolver
		if (viewModelType == null)
			viewModelType = _defaultViewToViewModelTypeResolver(view);

		// fallback to convention based
		if (viewModelType == null)
			viewModelType = _defaultViewTypeToViewModelTypeResolver(view.GetType());

		if (viewModelType == null)
			return;

		viewModel = _defaultViewModelFactoryWithViewParameter != null ? _defaultViewModelFactoryWithViewParameter(view, viewModelType) : _defaultViewModelFactory(viewModelType);
	}

	setDataContextCallback(view, viewModel);
}
```
```ad-note
You can change the convention to your liking. To change the ViewModelLocator naming convention, override the ConfigureViewModelLocator method in the App.xaml.cs class:
````csharp
protected override void ConfigureViewModelLocator()
{
    base.ConfigureViewModelLocator();

    ViewModelLocationProvider.SetDefaultViewTypeToViewModelTypeResolver((viewType) =>
    {
        var viewName = viewType.FullName.Replace(".ViewModels.", ".CustomNamespace.");
        var viewAssemblyName = viewType.GetTypeInfo().Assembly.FullName;
        var viewModelName = $"{viewName}ViewModel, {viewAssemblyName}";
        return Type.GetType(viewModelName);
    });
}
````
