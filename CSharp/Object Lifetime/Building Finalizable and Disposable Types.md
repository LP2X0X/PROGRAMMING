- On the one hand, you can use a finalizer. Using this technique, you have the peace of mind that comes with knowing the object cleans itself up when garbage collected (whenever that may be) without the need for user interaction. On the other hand, you can implement IDisposable to provide a way for the object user to clean up the object as soon as it is finished. However, if the caller forgets to call Dispose(), the unmanaged resources may be held in memory indefinitely.
- As you might suspect, it is possible to blend both techniques into a single class definition. By doing so, you gain the best of both models. If the object user does remember to call Dispose(), you can inform the garbage collector to bypass the finalization process by calling GC.SuppressFinalize(). If the object user forgets to call Dispose(), the object will eventually be finalized and have a chance to free up the internal resources.
- Microsoft defined a formal, prim-and-proper disposal pattern that strikes a balance between robustness, maintainability, and performance.
```csharp
class MyResourceWrapper : IDisposable  
{  
	// Used to determine if Dispose() has already been called.  
	private bool disposed = false;  
	
	public void Dispose()  
	{  
		// Call our helper method.  
		// Specifying "true" signifies that the object user triggered the cleanup.  
		CleanUp(true);  
		// Now suppress finalization.  
		GC.SuppressFinalize(this);  
	}  
	
	private void CleanUp(bool disposing)  
	{  
		// Be sure we have not already been disposed!  
		if (!this.disposed)  
		{  
			// If disposing equals true, dispose all managed resources.  
			if (disposing)  
			{  
				// Dispose managed resources.  
			}  
			// Clean up unmanaged resources here.  
		}  
		disposed = true;  
	}  
	
	~MyResourceWrapper()  
	{  
		// Call our helper method.  
		// Specifying "false" signifies that the GC triggered the cleanup.  
		CleanUp(false);  
	}  
}
```

```ad-important
Notice that MyResourceWrapper now defines a private helper method named CleanUp(). By specifying true as an argument, you indicate that the object user has initiated the cleanup, so you should clean up all managed and unmanaged resources. However, when the garbage collector initiates the cleanup, you specify false when calling CleanUp() to ensure that internal disposable objects are not disposed (as you can’t assume they are still in memory!). Last but not least, the bool member variable (disposed) is set to true before exiting CleanUp() to ensure that Dispose() can be called numerous times without error.
```