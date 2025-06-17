- The IRegionMemberLifetime interface defines a single read-only property, KeepAlive. If this property returns false, the view is removed from the region when it is deactivated. Because the region no longer has a reference to the view, it then becomes eligible for garbage collection (unless some other component in your application maintains a reference to it).
- You can implement this interface on your view or your view model classes.
```csharp
public class EmployeeDetailsViewModel : IRegionMemberLifetime
{
    public bool KeepAlive
    {
        get { return true; }
    }
}
```