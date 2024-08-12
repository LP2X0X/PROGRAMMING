- When defining properties, the access level for the get and set methods can be different. Revisiting the Social Security number, if the goal is to prevent the modification of the number from outside the class, then declare the get method as public but the set method as private, like this:  
```csharp
public string SocialSecurityNumber  
{  
	get => _empSSN;  
	private set => _empSSN = value;  
}  
```
- Note that this changes the property from read-only to read-write. The difference is that the write is hidden from anything outside the defining class.