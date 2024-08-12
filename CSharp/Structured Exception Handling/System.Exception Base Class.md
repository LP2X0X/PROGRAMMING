- All exceptions ultimately derive from the System.Exception base class, which in turn derives from System.Object.  Here is the main definition of this class (note that some of these members are virtual and may thus be overridden by derived classes):
```csharp
public class Exception : ISerializable  
{  
	// Public constructors  
	public Exception(string message, Exception innerException);  
	public Exception(string message);  
	public Exception();  
	...  
	// Methods  
	public virtual Exception GetBaseException();  
	public virtual void GetObjectData(SerializationInfo info,  
	StreamingContext context);
	// Properties  
	public virtual IDictionary Data { get; }  
	public virtual string HelpLink { get; set; }  
	public int HResult {get;set;}  
	public Exception InnerException { get; }  
	public virtual string Message { get; }  
	public virtual string Source { get; set; }  
	public virtual string StackTrace { get; }  
	public MethodBase TargetSite { get; }  
}
```

--- 

## Core members of the System.Exception Type
| System.Exception Property | Meaning in Life                                                                                                                                                                                                                              |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Data                      | This read-only property retrieves a collection of key-value pairs (represented by an object implementing IDictionary) that provide additional, programmer-defined information about the exception. By default, this collection is empty.     |
| HelpLink                  | This property gets or sets a URL to a help file or website describing the error in full detail.                                                                                                                                              |
| InnerException            | This read-only property can be used to obtain information about the previous exceptions that caused the current exception to occur. The previous exceptions are recorded by passing them into the constructor of the most current exception. |
| Message                   | This read-only property returns the textual description of a given error. The error message itself is set as a constructor parameter.                                                                                                        |
| Source                    | This property gets or sets the name of the assembly, or the object, that threw the current exception.                                                                                                                                        |
| StackTrace                | This read-only property contains a string that identifies the sequence of calls that triggered the exception. As you might guess, this property is useful during debugging or if you want to dump the error to an external error log.        |
| TargetSite                |    This read-only property returns a MethodBase object, which describes numerous details about the method that threw the exception (invoking ToString() will identify the method by name).                                                                                                                                                                                                                                          |
