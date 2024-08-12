### Order of Actions When `Dispose` is Called

1. **Check Disposal Status**: The `Dispose` method checks if the object has already been disposed to avoid redundant operations.
2. **Call Protected Dispose**: The `Dispose` method calls the protected `Dispose(bool disposing)` method, passing `true` to indicate that the method is being called explicitly by the user.
3. **Release Managed Resources**: Inside the `Dispose(bool disposing)` method, if `disposing` is `true`, the method releases managed resources. This typically involves calling the `Dispose` method on any `IDisposable` fields or properties.
4. **Release Unmanaged Resources**: The method then releases unmanaged resources, such as handles or memory allocated through native code.
5. **Mark Object as Disposed**: The object is marked as disposed to prevent further use and redundant disposal actions.
6. **Suppress Finalization**: The `Dispose` method calls `GC.SuppressFinalize(this)` to prevent the garbage collector from calling the finalizer, as resources have already been cleaned up.
7. **Complete Disposal**: The `Dispose` method completes, and control returns to the user.