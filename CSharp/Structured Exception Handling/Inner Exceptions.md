- When you encounter an exception while processing another exception, best practice states that you should record the new exception object as an “inner exception” within a new object of the same type as the initial exception.
```ad-warning
InnerException is read only, therefore doing the below will causes a compile error:
````csharp
e.InnerException = e2;
```

```csharp
//Update the exception handler  
catch (CarIsDeadException e)  
{  
	try  
	{  
		FileStream fs = File.Open(@"C:\carErrors.txt", FileMode.Open);  
		...  
	}  
	catch (Exception e2)  
	{  
		//This causes a compile error-InnerException is read only  
		//e.InnerException = e2;  
		// Throw an exception that records the new exception,  
		// as well as the message of the first exception.  
		throw new CarIsDeadException( e.CauseOfError, e.ErrorTimeStamp, e.Message, e2); }  
	}
}
```