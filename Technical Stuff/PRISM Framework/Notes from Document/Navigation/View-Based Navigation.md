- Although state-based navigation can be useful for the scenarios outlined earlier, navigation within an application will most often be accomplished by replacing one view within the application's UI with another. In Prism, this style of navigation is referred to as view-based navigation.
- Previous versions of Prism allowed views to be displayed in a region in two ways:
### View injection
- The first, called view injection, allows views to be programmatically displayed in a region. This approach is useful for dynamic content, where the view to be displayed in the region changes frequently, according to the application's presentation logic.
- View injection is supported through the Add method on the Region class:
```csharp
IRegionManager regionManager = ...;
IRegion mainRegion = regionManager.Regions["MainRegion"];
InboxView view = this.container.Resolve<InboxView>();
mainRegion.Add(view);
```
### View discovery
- The second method, called view discovery, allows a module to register a view type against a region name. Whenever a region with the specified name is displayed, an instance of the specified view will be automatically created and displayed in the region. This approach is useful for relatively static content, where the view to be displayed in a region does not change.
- View discovery is supported through the RegisterViewWithRegion method on the RegionManager class. This method allows you to specify a callback method that will be called when the named region is shown.
```csharp
IRegionManager regionManager = ...;
regionManager.RegisterViewWithRegion("MainRegion", () =>
				   container.Resolve<InboxView>());
//
IRegionManager regionManager = ...;
regionManager.RegisterViewWithRegion("MainRegion", typeof(InboxView));
```

---

## Basic region navigation
- Navigation within a region means that a new view is to be displayed within that region. The view to be displayed is identified via a URI, which, by default, refers to the name of the view to be created. You can programmatically initiate navigation using the **RequestNavigate** method defined by the INavigateAsync interface.
```csharp
IRegion mainRegion = ...;
mainRegion.RequestNavigate(new Uri("InboxView", UriKind.Relative));
```
```ad-note
RequestNavigate has many overload methods.
```
- You can also call the RequestNavigate method on the RegionManager, which allows you to specify the name of the region to be navigated.
```csharp
IRegionManager regionManager = ...;
regionManager.RequestNavigate("MainRegion",
                                new Uri("InboxView", UriKind.Relative));
```
- The **RequestNavigate** method also allows you to specify a callback method, or a delegate, which will be called when navigation is complete.
- The RequestNavigate method also allows you to specify a callback method, or a delegate, which will be called when navigation is complete.
```csharp
private void SelectedEmployeeChanged(object sender, EventArgs e)
{
    ...
    regionManager.RequestNavigate(RegionNames.TabRegion,
                        "EmployeeDetails", NavigationCompleted);
}
private void NavigationCompleted(NavigationResult result)
{
    ...
}
```
- The NavigationResult class defines properties that provide information about the navigation operation. The Result property indicates whether or not navigation succeeded. If navigation was successful, then the Result property will be true. If navigation failed, normally because of returning 'continuationCallBack(false)' in the IConfirmNavigationResult.ConfirmNavigationRequest method, then the Result property will be false. If navigation failed due to an exception, the Result property will be false and the Error property provides a reference to any exception that was thrown during navigation.