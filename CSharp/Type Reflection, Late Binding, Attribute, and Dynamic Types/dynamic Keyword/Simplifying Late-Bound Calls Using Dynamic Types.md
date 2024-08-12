- One instance where you might decide to use the dynamic keyword is when you are working with reflection services, specifically when making late-bound method calls. Earlier in this chapter, you saw a few examples of when this type of method call can be useful, most commonly when you are building some type of extensible application. At that time, you learned how to use the Activator.CreateInstance() method to create an object, for which you have no compile-time knowledge of (beyond its display name). You can then make use of the types of the System.Reflection namespace to invoke members via late binding. Recall the earlier example:

```csharp
static void CreateUsingLateBinding(Assembly asm)  
{  
	try  
	{  
		// Get metadata for the Minivan type.  
		Type miniVan = asm.GetType("CarLibrary.MiniVan");  
		// Create the Minivan on the fly.  
		object obj = Activator.CreateInstance(miniVan);  
		// Get info for TurboBoost.  
		MethodInfo mi = miniVan.GetMethod("TurboBoost");  
		// Invoke method ("null" for no parameters).  
		mi.Invoke(obj, null);  
	}  

	catch (Exception ex)  
	{  
		Console.WriteLine(ex.Message);  
	}  
}
```
- While this code works as expected, you might agree it is a bit clunky. You must manually make use of the MethodInfo class, manually query the metadata, and so forth. The following is a version of this same method, now using the C# dynamic keyword and the DLR:

```csharp
static void InvokeMethodWithDynamicKeyword(Assembly asm)  
{  
	try  
	{  
		// Get metadata for the Minivan type.  
		Type miniVan = asm.GetType("CarLibrary.MiniVan");  
		// Create the Minivan on the fly and call method!  
		dynamic obj = Activator.CreateInstance(miniVan);  
		obj.TurboBoost();  
	}  
	catch (Exception ex)  
	{  
		Console.WriteLine(ex.Message);  
	}  
}
```