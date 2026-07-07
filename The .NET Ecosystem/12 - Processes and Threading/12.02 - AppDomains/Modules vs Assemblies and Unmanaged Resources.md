---
tags:
 - csharp
 - appdomains
 - processes
---

# Modules vs Assemblies and Unmanaged Resources

## The Two Lists

When a .NET application runs, there are **two independent systems** tracking loaded code inside the same process:

1. **The OS Loader** tracks every DLL loaded into the process — these are called **modules**
2. **The CLR** tracks every managed DLL it loaded — these are called **assemblies**

Every assembly is also a module (the OS loaded it too), but not every module is an assembly. Native DLLs that the CLR didn't load are modules only.

| Scenario                                | `Process.Modules` | `GetAssemblies()` |
| --------------------------------------- | ----------------- | ----------------- |
| Your C# app (`MyApp.dll`)               | Yes               | Yes               |
| A NuGet package (`Newtonsoft.Json.dll`) | Yes               | Yes               |
| P/Invoke target (`opencv_world.dll`)    | Yes               | **No**            |
| Windows system DLL (`kernel32.dll`)     | Yes               | **No**            |


---

## Why the Distinction Matters

The CLR can only manage what it knows about. For managed assemblies, the CLR handles everything — loading types, JIT-compiling methods, garbage collecting objects. For native modules, the CLR has **zero visibility** into what they allocate or hold. It only knows the memory address of the function to call.


---

## How P/Invoke Works Under the Hood

When your C# code calls a native function via `[DllImport]`, the CLR doesn't "reference" or "manage" that native DLL. It delegates to the OS:

1. Your C# code calls `[DllImport("opencv_world.dll")]`
2. The CLR asks the **OS loader** (`LoadLibrary` on Windows) to load that DLL
3. The OS loads it into the process's memory and adds it to its module list
4. The CLR holds a **function pointer** to the specific native function
5. When you call the P/Invoke method, the CLR marshals your .NET types to native types, jumps to that pointer, and marshals the result back

```
+------------------------------------------+
|              Windows Process             |
|                                          |
|  +---CLR Runtime---+  +--OS Loader----+  |
|  |                 |  |               |  |
|  |  Assemblies:    |  |  Modules:     |  |
|  |  - MyApp.dll    |  |  - MyApp.dll  |  |
|  |  - System.dll   |  |  - System.dll |  |
|  |                 |  |  - kernel32   |  |
|  |  P/Invoke:      |  |  - opencv.dll |  |
|  |  "call addr X" -----> opencv.dll   |  |
|  +-----------------+  +---------------+  |
+------------------------------------------+
```

The CLR and the OS loader are two independent systems that share the same process space. The CLR manages assemblies. The OS tracks all loaded DLLs as modules. Managed DLLs appear in **both** lists. Native DLLs appear only in the OS's list.


---

## What "Releasing an Unmanaged Resource" Actually Means

This connects directly to the module/assembly split. When you "release an unmanaged resource" in `Dispose()`, you're **calling a native OS function** that tells the OS to free something it allocated. You're not freeing memory yourself — you're telling the OS "I'm done with this."

### Concrete Example — File Handle

```csharp
public class MyFileReader : IDisposable
{
    private IntPtr _fileHandle; // just a number — an OS handle

    public MyFileReader(string path)
    {
        // P/Invoke -> asks OS to open the file
        // OS opens it, locks it, returns a handle (e.g., 0x0A4C)
        _fileHandle = NativeMethods.CreateFile(path, ...);
    }

    public void Dispose()
    {
        // P/Invoke -> tells OS "I'm done with handle 0x0A4C"
        // OS closes the file, releases the lock, frees its internal tracking
        NativeMethods.CloseHandle(_fileHandle);
        _fileHandle = IntPtr.Zero;
    }
}
```

### Step-by-Step: What Happens

| Step                       | Who does it | What happens                                                           |
| -------------------------- | ----------- | ---------------------------------------------------------------------- |
| `CreateFile()`             | OS (kernel) | OS opens file, creates internal tracking struct, returns handle number |
| Your object holds `IntPtr` | CLR         | Just stores an integer (the handle) on the managed heap — ~8 bytes     |
| `CloseHandle()` in Dispose | OS (kernel) | OS destroys its internal tracking, releases the file lock              |
| GC collects your object    | CLR         | Frees the ~8 bytes of managed memory                                   |

### Why Dispose Exists

The GC only knows about the 8-byte `IntPtr` on the managed heap. It has **zero visibility** into what the OS allocated behind that handle — the file lock, the kernel buffer, the internal data structures. The GC can't clean those up because it doesn't even know they exist.

`Dispose()` is the bridge between the CLR world and the OS world. You're not "freeing memory" in the .NET sense — you're telling the OS to clean up **its** side of things.

```ad-note
The `Finalizer` (`~MyFileReader()`) exists as a safety net for exactly this reason — if you forget to call `Dispose()`, the GC will eventually call the finalizer, which should call the same native cleanup function. But finalizers run on an unpredictable schedule, so the OS resource stays locked longer than necessary. See [[Dispose Pattern]].
```


---

## The Full Picture

```
        CLR World                           OS World
  (managed, GC tracks)              (unmanaged, OS tracks)
                          
  +------------------+               +------------------+
  |  MyFileReader    |               |  File Handle     |
  |  object on heap  |   Dispose()   |  kernel struct   |
  |                  | ------------> |  file lock       |
  |  _fileHandle: 42 |  CloseHandle  |  I/O buffer      |
  +------------------+               +------------------+
         |                                    |
         v                                    v
      GC frees                        OS frees on
      ~8 bytes                        CloseHandle()
```

The managed object is trivially small. The **real** resource lives in the OS. That's why "releasing unmanaged resources" means calling back into native code to tell the OS to let go.
