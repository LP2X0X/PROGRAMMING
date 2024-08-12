In .NET, a finalizable object is an object that can perform cleanup actions before the garbage collector reclaims its memory. This is typically done by overriding the `Finalize` method, which is often implemented through a destructor in C#. Finalization allows an object to release unmanaged resources like file handles, database connections, or other system resources that the garbage collector does not handle automatically.
The object is marked as **finalizable** if it supports a custom Finalize() method, and a pointer to this object is stored on an internal queue named the **finalization queue**. The finalization queue is a table maintained by the garbage collector that points to every object that must be finalized before it is removed from the heap.

```ad-note
When the garbage collector determines it is time to free an object from memory, it examines each entry on the finalization queue and copies the object off the heap to yet another managed structure termed the **finalization reachable table** (often abbreviated as freachable and pronounced “eff-reachable”). At this point, a separate thread is spawned to invoke the Finalize() method for each object on the freachable table at the next garbage collection. Given this, it will take, at the least, two garbage collections to truly finalize an object.
```

### Key Concepts

1. **Finalizer**: 
    - *A finalizer, also known as a destructor in C#, is a special method that is called by the garbage collector before an object is reclaimed. It allows the object to clean up unmanaged resources.*
    - Syntax in C#: `~ClassName() { // cleanup code }`

2. **Garbage Collection (GC)**:
    - The .NET garbage collector is responsible for automatically managing the allocation and release of managed objects. However, it does not handle unmanaged resources, which is where finalization comes into play.

3. **Dispose Pattern**:
    - The `IDisposable` interface and the `Dispose` method provide a more deterministic way to release unmanaged resources.
    - Typically, classes that implement `IDisposable` also implement a finalizer to ensure resources are released even if `Dispose` is not called.

### Example

Here is an example of a class that implements both `IDisposable` and a finalizer:

```csharp
public class ResourceHolder : IDisposable
{
    // Unmanaged resource
    private IntPtr unmanagedResource;
    
    // Managed resource
    private SomeManagedResource managedResource;

    // Constructor
    public ResourceHolder()
    {
        // Allocate resources
        unmanagedResource = /* allocation code */;
        managedResource = new SomeManagedResource();
    }

    // Finalizer (destructor)
    ~ResourceHolder()
    {
        // Finalizer calls Dispose with false indicating it's called by the GC
        Dispose(false);
    }

    // Public implementation of Dispose pattern callable by consumers
    public void Dispose()
    {
        Dispose(true);
        // Suppress finalization to avoid the overhead of calling the finalizer
        GC.SuppressFinalize(this);
    }

    // Protected implementation of Dispose pattern
    protected virtual void Dispose(bool disposing)
    {
        if (disposing)
        {
            // Free managed resources
            if (managedResource != null)
            {
                managedResource.Dispose();
                managedResource = null;
            }
        }

        // Free unmanaged resources
        if (unmanagedResource != IntPtr.Zero)
        {
            // Free the unmanaged resource
            unmanagedResource = IntPtr.Zero;
        }
    }
}
```

```ad-warning
It is illegal to override Finalize() on structure types. This makes perfect sense given that structures are value types, which are never allocated on the heap to begin with and, therefore, are not garbage collected!
```

### Best Practices

1. **Use `IDisposable` and `Dispose`**:
    - Implement `IDisposable` for classes that hold unmanaged resources.
    - Call the `Dispose` method to release resources explicitly when they are no longer needed.

2. **Suppress Finalization**:
    - Use `GC.SuppressFinalize(this)` in the `Dispose` method to prevent the finalizer from being called if resources have already been released.

3. **Avoid Finalizers if Possible**:
    - Finalizers add overhead to garbage collection. Use them only when necessary.
    - Prefer safe handle classes from the .NET Framework (`SafeHandle`) to manage unmanaged resources without needing a finalizer.

By following these guidelines and using the dispose pattern correctly, you can manage resource cleanup efficiently and avoid common pitfalls associated with finalizable objects in .NET.