- The System.Exception.TargetSite property allows you to determine various details about the method that threw a given exception. Printing the value of TargetSite will display the return type, name, and parameter types of the method that threw the exception. However, TargetSite does not return just a vanilla-flavored string but rather a strongly typed System.Reflection. MethodBase object. This type can be used to gather numerous details regarding the offending method, as well as the class that defines the offending method.
```csharp
// TargetSite actually returns a MethodBase object.  
catch(Exception e)  
{  
	Console.WriteLine("\n*** Error! ***");  
	Console.WriteLine("Member name: {0}", e.TargetSite);  
	Console.WriteLine("Class defining member: {0}", e.TargetSite.DeclaringType);  
	Console.WriteLine("Member type: {0}", e.TargetSite.MemberType);  
	Console.WriteLine("Message: {0}", e.Message);  
	Console.WriteLine("Source: {0}", e.Source);  
}  
Console.WriteLine("\n***** Out of exception logic *****");  
Console.ReadLine();
```
- This time, you use the MethodBase.DeclaringType property to determine the fully qualified name of the class that threw the error (SimpleException.Car, in this case) as well as the MemberType property of the MethodBase object to identify the type of member (such as a property versus a method) where this exception originated.