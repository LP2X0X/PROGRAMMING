- Notice that the description of Motorcycle is assigned a description using a new bit of attribute syntax termed a named property. In the constructor of the first \[VehicleDescription\] attribute, you set the underlying string data by using the Description property. If this attribute is reflected upon by an external agent, the value is fed into the Description property (named property syntax is legal only if the attribute supplies a writable .NET property).

```csharp
// Assign description using a "named property."  
[VehicleDescription(Description = "My rocking Harley")]  
public class Motorcycle  
{

}
```