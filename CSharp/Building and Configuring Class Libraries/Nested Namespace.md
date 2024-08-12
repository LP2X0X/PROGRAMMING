```ad-important
A common practice is to group files in a namespace by directory. Where a file lives in the directory structure has no impact on namespaces. however, it does make the namespace structure clearer to other developers. therefore, many developers and code linting tools expect the namespaces to match the folder structures.
```
### Option 1:
```csharp
namespace CustomNamespaces  
{  
	namespace MyShapes  
	{  
		// Circle class  
		public class Circle  
		{  
		/* Interesting methods... */  
		}  
	}  
}
```

### Option 2:
```csharp
namespace CustomNamespaces;  
namespace MyShapes  
{  
	// Circle class  
	public class Circle  
	{  
	/* Interesting methods... */  
	}  
}
```

### Option 3:
```csharp
namespace CustomNamespaces.MyShapes;  
// Circle class  
public class Circle  
{  
/* Interesting methods... */  
}
```