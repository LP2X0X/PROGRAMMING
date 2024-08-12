Implementing the `IDisposable` interface to dispose of unmanaged resources involves creating a class that includes unmanaged resources (e.g., file handles, database connections, or custom native resources). Here’s a detailed example of how to implement this pattern:

### Example: Implementing `IDisposable` for Unmanaged Resources

Suppose we have a class that wraps an unmanaged resource, such as a native Windows handle. For demonstration purposes, let's assume we have a simple unmanaged resource represented by an `IntPtr`.

#### Unmanaged Resource Simulation

In a real-world scenario, this could be a handle to a file, a network connection, or some other resource that needs to be explicitly released.

#### ResourceHolder Class

Here’s how to implement `IDisposable` to properly clean up this resource:

```csharp
using System;
using System.Runtime.InteropServices;

public class ResourceHolder : IDisposable
{
    // Unmanaged resource.
    private IntPtr _unmanagedResource;
    // Track whether Dispose has been called.
    private bool _disposed = false;

    public ResourceHolder()
    {
        // Allocate the unmanaged resource.
        _unmanagedResource = AllocateUnmanagedResource();
    }

    // Public implementation of Dispose pattern callable by consumers.
    public void Dispose()
    {
        Dispose(true);
        // Suppress finalization.
        GC.SuppressFinalize(this);
    }

    // Protected implementation of Dispose pattern.
    protected virtual void Dispose(bool disposing)
    {
        if (_disposed)
            return;

        if (disposing)
        {
            // Free other state (managed objects).
        }

        // Free your own state (unmanaged objects).
        FreeUnmanagedResource(_unmanagedResource);
        _unmanagedResource = IntPtr.Zero;

        // Set large fields to null.

        _disposed = true;
    }

    // Destructor to ensure resources are freed if Dispose is not called.
    ~ResourceHolder()
    {
        Dispose(false);
    }

    private IntPtr AllocateUnmanagedResource()
    {
        // Simulate allocation of an unmanaged resource.
        return Marshal.AllocHGlobal(100); // Allocate 100 bytes.
    }

    private void FreeUnmanagedResource(IntPtr resource)
    {
        if (resource != IntPtr.Zero)
        {
            // Simulate releasing an unmanaged resource.
            Marshal.FreeHGlobal(resource);
        }
    }
}
```

### Explanation:

1. **Unmanaged Resource**: 
   - `_unmanagedResource` is an `IntPtr` representing an unmanaged resource.

2. **Constructor**: 
   - The constructor allocates the unmanaged resource.

3. **Dispose Method**: 
   - `Dispose()` is the public method that consumers call to release resources. It calls the protected `Dispose(bool disposing)` method and suppresses finalization to avoid the destructor running.

4. **Dispose Pattern**: 
   - `Dispose(bool disposing)` is protected and virtual, allowing derived classes to override it. It checks if the object has already been disposed to prevent multiple disposals.
   - If `disposing` is `true`, this means the method was called by consumer code and managed resources should be disposed of.
   - Unmanaged resources are always freed regardless of the value of `disposing`.

5. **Finalizer**: 
   - The finalizer (`~ResourceHolder`) calls `Dispose(false)` to ensure that unmanaged resources are freed if `Dispose` is not called. This acts as a safety net to clean up resources.

6. **AllocateUnmanagedResource**: 
   - Simulates the allocation of an unmanaged resource using `Marshal.AllocHGlobal`.

7. **FreeUnmanagedResource**: 
   - Simulates the release of an unmanaged resource using `Marshal.FreeHGlobal`.

### Usage:

To ensure that the `Dispose` method is called when you are done with the `ResourceHolder` instance, you should use a `using` statement or explicitly call `Dispose` in a `try/finally` block:

```csharp
using (ResourceHolder resourceHolder = new ResourceHolder())
{
    // Use the resource.
}

// At this point, Dispose has been called automatically.
```

Or:

```csharp
ResourceHolder resourceHolder = null;
try
{
    resourceHolder = new ResourceHolder();
    // Use the resource.
}
finally
{
    if (resourceHolder != null)
    {
        resourceHolder.Dispose();
    }
}
```

This pattern ensures that unmanaged resources are properly released, preventing resource leaks and ensuring the efficient use of system resources.