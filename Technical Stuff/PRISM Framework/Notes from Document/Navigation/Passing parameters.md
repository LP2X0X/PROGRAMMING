- To implement the required navigational behavior in your application, you will often need to specify additional data during navigation request than just the target view name. The NavigationContext object provides access to the navigation URI, and to any parameters that were specified within it or externally. You can access the NavigationContext from within the IsNavigationTarget, OnNavigatedFrom, and OnNavigatedTo methods.
- Prism provides the NavigationParameters class to help specify and retrieve navigation parameters. The NavigationParameters class maintains a list of name-value pairs, one for each parameter. You can use this class to pass parameters as part of navigation URI or for passing object parameters.
- Pass as part of URI:
```csharp
Employee employee = Employees.CurrentItem as Employee;
if (employee != null)
{
    var navigationParameters = new NavigationParameters();
    navigationParameters.Add("ID", employee.Id);
    _regionManager.RequestNavigate(RegionNames.TabRegion,
            new Uri("EmployeeDetailsView" + navigationParameters.ToString(), UriKind.Relative));
}
```
- Pass as an parameter:
```csharp
Employee employee = Employees.CurrentItem as Employee;
if (employee != null)
{
    var parameters = new NavigationParameters();
    parameters.Add("ID", employee.Id);
    parameters.Add("myObjectParameter", new ObjectParameter());
    regionManager.RequestNavigate(RegionNames.TabRegion,
            new Uri("EmployeeDetailsView", UriKind.Relative), parameters);
}
```
- You can retrieve the navigation parameters using the Parameters property on the NavigationContext object:
```csharp
public void OnNavigatedTo(NavigationContext navigationContext)
{
    string id = navigationContext.Parameters["ID"];
    ObjectParameter myParameter = navigationContext.Parameters["myObjectParameter"];
}
```