```ad-note
Golden Rule: Allocate a class instance onto the managed heap using the new keyword and forget about it.
```

```ad-note
If the managed heap does not have sufficient memory to allocate a requested object, a garbage collection will occur.
```

```ad-note
The only compelling reason to override Finalize() is if your C# class is using unmanaged resources via pinvoke or complex COm interoperability tasks (typically via various members defined by the System.Runtime.InteropServices.Marshal type). The reason is that under these scenarios you are manipulating memory that the runtime cannot manage.
```