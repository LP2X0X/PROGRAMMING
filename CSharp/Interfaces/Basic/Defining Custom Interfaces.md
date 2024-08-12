```csharp
namespace CustomInterfaces;  
// This interface defines the behavior of "having points."  

public interface IPointy  
{  
	// Implicitly public and abstract.  
	byte GetNumberOfPoints();  
}
```

```ad-warning
Interfaces in C# 8 cannot define data fields or nonstatic constructors.
```

---

- Interface types are also able to define any number of property prototypes.
```csharp
// The pointy behavior as a read-only property.  
public interface IPointy  
{  
	// Implicitly public and abstract.  
	//byte GetNumberOfPoints();  
	
	// A read-write property in an interface would look like:  
	//string PropName { get; set; }  
	
	// while a read-only property in an interface would be:  
	byte Points { get; }  
}
```