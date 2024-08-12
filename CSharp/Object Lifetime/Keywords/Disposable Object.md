In .NET, a disposable object is an object that implements the `IDisposable` interface to provide a standardized way to release unmanaged resources deterministically. The `IDisposable` interface defines a single method, `Dispose`, which is intended to be called by consumers of the object to explicitly free resources when they are no longer needed.
```ad-note
When you do implement the IDisposable interface, the assumption is that when the object user is finished using the object, the object user manually calls Dispose() before allowing the object reference to drop out of scope. In this way, an object can perform any necessary cleanup of unmanaged resources without incurring the hit of being placed on the finalization queue and without waiting for the garbage collector to trigger the class’s finalization logic.
```

```ad-note
Notice that a Dispose() method not only is responsible for releasing the type’s unmanaged resources but can also call Dispose() on any other contained disposable methods. Unlike with Finalize(), it is perfectly safe to communicate with other managed objects within a Dispose() method. The reason is simple: the garbage collector has no clue about the IDisposable interface and will never call Dispose(). Therefore, when the object user calls this method, the object is still living a productive life on the managed heap and has access to all other heap-allocated objects.
```

### Key Concepts

1. **IDisposable Interface**:
    - Defines the `Dispose` method for releasing unmanaged resources.
    - Syntax: `public interface IDisposable { void Dispose(); }`

2. **Dispose Method**:
    - Contains logic to release unmanaged resources and other cleanup code.
    - Should be called explicitly by the consumer to free resources as soon as they are no longer needed.

3. **Using Statement**:
    - Provides a convenient syntax to ensure `Dispose` is called automatically.
    - Syntax: `using (var resource = new Resource()) { /* Use resource */ }`

4. **Finalization (Optional)**:
    - Implementing a finalizer allows cleanup if `Dispose` is not called.
    - Suppress finalization using `GC.SuppressFinalize` within the `Dispose` method.

```ad-note
A Dispose() method not only is responsible for releasing the type’s unmanaged resources but can also call Dispose() on any other contained disposable methods.
```
### Example

Here is an example of a class that implements `IDisposable`:

```csharp
using System;

public class ResourceHolder : IDisposable
{
    // Unmanaged resource
    private IntPtr unmanagedResource;

    // Managed resource
    private SomeManagedResource managedResource;

    // Flag to detect redundant calls
    private bool disposed = false;

    // Constructor
    public ResourceHolder()
    {
        // Allocate resources
        unmanagedResource = AllocateUnmanagedResource();
        managedResource = new SomeManagedResource();
    }

    // Finalizer (destructor)
    ~ResourceHolder()
    {
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
        if (!disposed)
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
                FreeUnmanagedResource(unmanagedResource);
                unmanagedResource = IntPtr.Zero;
            }

            disposed = true;
        }
    }

    // PInvoke to allocate unmanaged resource
    [DllImport("UnmanagedLibrary.dll", CallingConvention = CallingConvention.Cdecl)]
    private static extern IntPtr AllocateUnmanagedResource();

    // PInvoke to free unmanaged resource
    [DllImport("UnmanagedLibrary.dll", CallingConvention = CallingConvention.Cdecl)]
    private static extern void FreeUnmanagedResource(IntPtr resource);
}
```

### Explanation

1. **Resource Allocation**:
    - The constructor allocates both unmanaged and managed resources.

2. **Dispose Method**:
    - The `Dispose` method provides a way for consumers to release resources explicitly.
    - It calls a protected `Dispose(bool disposing)` method to handle both managed and unmanaged resource cleanup.

3. **Finalizer**:
    - The finalizer (`~ResourceHolder`) ensures resources are cleaned up if `Dispose` is not called.
    - The finalizer calls `Dispose(false)` to release only unmanaged resources, as managed resources might have already been collected by the GC.

4. **Dispose Pattern**:
    - The `Dispose(bool disposing)` method differentiates between explicit disposal (when `Dispose` is called) and implicit disposal (by the finalizer).
    - The flag `disposed` prevents redundant calls to `Dispose`.

5. **Suppress Finalization**:
    - `GC.SuppressFinalize(this)` is called within the `Dispose` method to avoid the overhead of finalization if the resources have already been released explicitly.

### Usage

Using the `ResourceHolder` class with the `using` statement ensures that `Dispose` is called automatically:

```csharp
using (var resource = new ResourceHolder())
{
    // Use the resource
}
// Dispose is called automatically at the end of the using block
```

By following this pattern, you ensure that unmanaged resources are released promptly and efficiently, avoiding resource leaks and ensuring better application performance and reliability.