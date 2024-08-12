If you want to configure your property as a write-only property, omit the get block, like this:  
```csharp
public int Id  
{  
	set { _empId = value; }  
}
```
