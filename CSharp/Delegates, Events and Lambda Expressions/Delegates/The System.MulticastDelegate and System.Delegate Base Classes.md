- So, when you build a type using the C# delegate keyword, you are indirectly declaring a class type that derives from System.MulticastDelegate. This class provides descendants with access to a list that contains the addresses of the methods maintained by the delegate object, as well as several additional methods (and a few overloaded operators) to interact with the invocation list.
- Here are some select members of System.MulticastDelegate:

```csharp
public abstract class MulticastDelegate : Delegate  
{  
	// Returns the list of methods "pointed to."  
	public sealed override Delegate[] GetInvocationList();  
	// Overloaded operators.  
	public static bool operator ==  
	(MulticastDelegate d1, MulticastDelegate d2);  
	public static bool operator !=  
	(MulticastDelegate d1, MulticastDelegate d2);  
	// Used internally to manage the list of methods maintained by the delegate.  
	private IntPtr _invocationCount;  
	private object _invocationList;  
}
```

- System.MulticastDelegate obtains additional functionality from its parent class, System.Delegate. Here is a partial snapshot of the class definition:

```csharp
public abstract class Delegate : ICloneable, ISerializable  
{  
	// Methods to interact with the list of functions.  
	public static Delegate Combine(params Delegate[] delegates);  
	public static Delegate Combine(Delegate a, Delegate b);  
	public static Delegate Remove(  
	Delegate source, Delegate value);  
	public static Delegate RemoveAll(  
	Delegate source, Delegate value);  
	
	// Overloaded operators.  
	public static bool operator ==(Delegate d1, Delegate d2);  
	public static bool operator !=(Delegate d1, Delegate d2);  
	
	// Properties that expose the delegate target.  
	public MethodInfo Method { get; }  
	public object Target { get; }  
}
```

![[Pasted image 20240618143403.png|center|700]]