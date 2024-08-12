```csharp
public string SocialSecurityNumber  
{  
	get { return _empSSN; }  
}
```

or 

```csharp
public string SocialSecurityNumber => _empSSN;
```
- Unless you are willing to redesign the property as read-write (which you will do soon), your only choice with read-only properties would be to use the underlying member variable within your constructor logic.