- Exceptions that are thrown by the .NET platform are (appropriately) called system exceptions. These exceptions are generally regarded as nonrecoverable, fatal errors. 
```ad-important
System exceptions derive directly from a base class named System.SystemException, which in turn derives from System.Exception (which derives from System.Object).
```

```csharp
public class SystemException : Exception  
{  
// Various constructors.  
}
```

- Given that the System.SystemException type does not add any functionality beyond a set of custom constructors, you might wonder why SystemException exists in the first place. Simply put, when an exception type derives from System.SystemException, **you are able to determine that the .NET runtime is the entity that has thrown the exception, rather than the code base of the executing application**.