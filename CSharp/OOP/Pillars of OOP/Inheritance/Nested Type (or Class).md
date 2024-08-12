- Understanding why you would want to do this might not be readily apparent. To understand this technique, ponder the following traits of nesting a type:  
	- Nested types allow you to gain complete control over the access level of the inner type because they may be declared privately (recall that non-nested classes cannot be declared using the private keyword).  
	- Because a nested type is a member of the containing class, it can access private members of the containing class.  
	- Often, a nested type is useful only as a helper for the outer class and is not intended for use by the outside world.
```csharp
public class OuterClass  
{  
	// A public nested type can be used by anybody.  
	public class PublicInnerClass {}  
	// A private nested type can only be used by members  
	// of the containing class.  
	private class PrivateInnerClass {}  
}
```

- When a type nests another class type, it can create member variables of the type, just as it would for any point of data. However, if you want to use a nested type from outside the containing type, you must qualify it by the scope of the nesting type.
```csharp
// Create and use the public inner class. OK!  
OuterClass.PublicInnerClass inner;  
inner = new OuterClass.PublicInnerClass();  
// Compiler Error! Cannot access the private class.  
OuterClass.PrivateInnerClass inner2;  
inner2 = new OuterClass.PrivateInnerClass();
```