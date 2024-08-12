- Frequently, the views and view models in your application will want to participate in navigation. The INavigationAware interface enables this. You can implement this interface on the view or (more commonly) the view model. By implementing this interface, your view or view model can opt-in to participate in the navigation process.
- This interface allows the view or view model to participate in a navigation operation. The INavigationAware interface defines three methods.
```csharp
public interface INavigationAware
{
    bool IsNavigationTarget(NavigationContext navigationContext);
    void OnNavigatedTo(NavigationContext navigationContext);
    void OnNavigatedFrom(NavigationContext navigationContext);
}
```
- The **IsNavigationTarget** method allows an existing (displayed) view or view model to indicate whether it is able to handle the navigation request. This is useful in cases where you can re-use an existing view to handle the navigation operation or when navigating to a view that already exists. If any of them return true, Prism will use that existing view model for navigation, avoiding the creation of a new instance. If all of them return false, a new instance will be created. For example, a view displaying customer information can be updated to display a different customer's information.
- The OnNavigatedFrom and OnNavigatedTo methods are called during a navigation operation. If the currently active view in the region implements this interface (or its view model), its OnNavigatedFrom method is called before navigation takes place. The **OnNavigatedFrom** method allows the previous view to save any state or to prepare for its deactivation or removal from the UI, for example, to save any changes that the user has made to a web service or database.
- If the newly created view implements this interface (or its view model), its **OnNavigatedTo** method is called after navigation is complete. The OnNavigatedTo method allows the newly displayed view to initialize itself, potentially using any parameters passed to it on the navigation URI.