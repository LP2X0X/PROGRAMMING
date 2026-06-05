---
tags:
 - csharp
 - assemblies
 - interop
---

## Introduction

When C# code calls a native function via `[DllImport]`, the data you pass does not simply teleport across the boundary. The managed world (CLR heap, garbage-collected objects, UTF-16 strings) and the unmanaged world (raw pointers, C-style structs, null-terminated `char*`) represent the same logical data in fundamentally different ways. **Marshalling** is the process by which the CLR converts data from one representation to the other so that both sides can read it correctly.

Understanding marshalling is essential for P/Invoke work because it determines:

- Whether your native call is fast (zero-copy) or slow (allocate-convert-copy)
- Whether your struct layouts match what the native API expects
- Whether your strings arrive as ANSI, UTF-8, or wide characters
- Whether your callback delegates survive garbage collection

This note covers the full marshalling story: when it happens, the critical blittable vs non-blittable distinction, common scenarios with code, the `[MarshalAs]` attribute, the `Marshal` class for manual control, and performance implications.

For the structure of .NET assemblies themselves, see [[Format of a .NET Assembly]]. For why assemblies exist and what roles they play, see [[The Role of .NET Assemblies]].

---

## What Is Marshalling

**Marshalling** is the conversion of data between managed (CLR) and unmanaged (native) memory representations when that data crosses the interop boundary.

The need arises because managed and unmanaged code represent even simple data differently:

| Concept | Managed (C#) | Unmanaged (C/C++) |
|---|---|---|
| **String** | `System.String` -- an object on the GC heap, UTF-16, length-prefixed, immutable | `char*` (ANSI) or `wchar_t*` (wide) -- raw pointer to null-terminated buffer |
| **Boolean** | `bool` -- 1 or 4 bytes depending on context | `BOOL` -- 4-byte `int` (0 or non-zero) in Win32 |
| **Character** | `char` -- 2 bytes, UTF-16 | `char` -- 1 byte, ANSI |
| **Array** | Managed array object with bounds checking | Raw pointer to contiguous memory |
| **Struct** | May have padding, GC references, reordered fields | Fixed layout, no GC, predictable padding |

When you call a native function through `[DllImport]`, the CLR's **interop marshaller** automatically handles these conversions. But you can also marshal manually using the `Marshal` class when you need precise control.

```ad-note
Marshalling is not unique to C#. Every language that bridges managed and native code faces this problem -- Java's JNI, Python's ctypes, and Rust's FFI all have equivalent mechanisms. The CLR's marshalling layer is one of the most mature and configurable.
```

---

## When Marshalling Happens

Marshalling occurs at three distinct points in the interop lifecycle:

### 1. Automatic Marshalling on P/Invoke Calls

Every `[DllImport]` call triggers the marshalling pipeline. The CLR intercepts the call, converts each argument from its managed representation to its native equivalent, invokes the native function, then converts the return value back.

```
                        MANAGED WORLD                    UNMANAGED WORLD
                    ┌─────────────────┐              ┌─────────────────┐
                    │   C# code       │              │  Native DLL     │
                    │   calls         │              │  function       │
                    │   NativeFunc()  │              │  NativeFunc()   │
                    └───────┬─────────┘              └───────▲─────────┘
                            │                                │
                            ▼                                │
                    ┌─────────────────┐              ┌───────┴─────────┐
                    │  Marshal args   │──────────────►│  Execute native │
                    │  C# → native    │              │  code           │
                    └─────────────────┘              └───────┬─────────┘
                                                             │
                    ┌─────────────────┐                      │
                    │  Marshal return  │◄─────────────────────┘
                    │  native → C#    │
                    └───────┬─────────┘
                            │
                            ▼
                    ┌─────────────────┐
                    │  Continue C#    │
                    │  execution      │
                    └─────────────────┘
```

### 2. Callbacks from Native Code into Managed Code

When you pass a delegate to a native function as a callback (a function pointer), the CLR creates a **thunk** -- a small native-callable wrapper that marshals the callback's arguments from native to managed, invokes the delegate, then marshals the return value back.

### 3. Manual Marshalling via the Marshal Class

When automatic marshalling is not flexible enough, you call `Marshal` class methods directly to allocate unmanaged memory, copy data across the boundary, and convert types manually.

```ad-note
In practice, most P/Invoke calls use automatic marshalling. You reach for manual marshalling when dealing with complex or nested native structures, when you need to read from a pointer returned by native code, or when you need to manage the lifetime of unmanaged memory explicitly.
```

---

## Blittable vs Non-Blittable Types

This is the single most important concept for P/Invoke performance and correctness. The distinction determines whether the CLR can pass your data by pointer (fast) or must allocate, convert, and copy it (slow).

### Blittable Types

A **blittable type** has the *exact same bit-for-bit representation* in managed and unmanaged memory. No conversion is needed. The CLR simply **pins** the managed object in place (so the GC will not move it) and passes a raw pointer directly to native code.

This means:
- **Zero allocation** of temporary unmanaged memory
- **Zero copying** of data
- **Zero conversion** overhead
- The native function reads/writes the *same bytes* the CLR sees

The blittable primitive types are:

| Type | Size | Notes |
|---|---|---|
| `byte` / `sbyte` | 1 byte | |
| `short` / `ushort` | 2 bytes | |
| `int` / `uint` | 4 bytes | |
| `long` / `ulong` | 8 bytes | |
| `float` | 4 bytes | IEEE 754 single |
| `double` | 8 bytes | IEEE 754 double |
| `IntPtr` / `UIntPtr` | 4 or 8 bytes | Platform-dependent pointer size |

A **struct** is blittable if *all* of its fields are blittable types and it uses `LayoutKind.Sequential` or `LayoutKind.Explicit`:

```csharp
// This struct is blittable -- all fields are blittable primitives
[StructLayout(LayoutKind.Sequential)]
public struct Point
{
    public int X;
    public int Y;
}
```

```ad-warning
A one-dimensional array of blittable types (e.g., `int[]`, `double[]`) is blittable *for marshalling purposes* -- the CLR pins it and passes a pointer to the first element. But the array *object itself* is a managed reference type. The distinction matters: you can pass `int[]` efficiently, but you cannot embed a managed array inside a blittable struct.
```

### Non-Blittable Types

A **non-blittable type** has a *different* representation in managed vs unmanaged memory. The CLR must:

1. **Allocate** temporary unmanaged memory
2. **Convert** the managed data to the native format
3. **Copy** the converted data to the unmanaged buffer
4. **Call** the native function with a pointer to the buffer
5. **Convert back** the result (if it is an out/ref parameter or return value)
6. **Free** the temporary unmanaged memory

The common non-blittable types:

| Type | Why It Is Non-Blittable |
|---|---|
| `string` | Managed: GC heap object, UTF-16, length-prefixed. Native: null-terminated `char*` or `wchar_t*` |
| `bool` | Managed: 1 byte (`true`/`false`). Native Win32 `BOOL`: 4-byte `int`. The sizes differ. |
| `char` | Managed: 2 bytes (UTF-16). Native: 1 byte (ANSI) by default |
| `object` | No unmanaged equivalent |
| `class` (reference types) | Contains an object header and method table pointer that have no native meaning |
| Delegates | Must be converted to native function pointers |
| Arrays of non-blittable types | Each element requires individual conversion |

```ad-danger
**The `bool` trap.** Many developers assume `bool` is blittable because it seems like a simple value type. It is not. C# `bool` is 1 byte; Win32 `BOOL` is 4 bytes. If you put a `bool` in a P/Invoke struct without `[MarshalAs]`, you will get silent data corruption from misaligned fields. Always use `[MarshalAs(UnmanagedType.Bool)]` (4-byte) or `[MarshalAs(UnmanagedType.U1)]` (1-byte) to be explicit.
```

### Quick Decision Flowchart

```
Is the type a primitive numeric type (byte, int, float, etc.)?
├── YES → Blittable ✓
└── NO
    Is it IntPtr or UIntPtr?
    ├── YES → Blittable ✓
    └── NO
        Is it a struct with ONLY blittable fields + Sequential/Explicit layout?
        ├── YES → Blittable ✓
        └── NO → Non-Blittable ✗ (marshalling overhead applies)
```

---

## Common Marshalling Scenarios

### Strings

C# `string` is the most frequently marshalled type and the one with the most pitfalls. The core problem: C# strings are immutable UTF-16 objects on the managed heap. Native code expects a null-terminated pointer to either ANSI or wide characters.

**Controlling character encoding with `CharSet`:**

```csharp
// ANSI -- CLR converts UTF-16 → ANSI, allocates a temporary char* buffer
[DllImport("user32.dll", CharSet = CharSet.Ansi)]
static extern int MessageBoxA(IntPtr hWnd, string text, string caption, uint type);

// Unicode (wide) -- CLR can pin the string and pass wchar_t* directly
// because C# strings are already UTF-16, matching wchar_t on Windows
[DllImport("user32.dll", CharSet = CharSet.Unicode)]
static extern int MessageBoxW(IntPtr hWnd, string text, string caption, uint type);
```

```ad-note
When `CharSet = CharSet.Unicode`, string marshalling can be *almost* as fast as blittable types because the CLR pins the string's internal buffer and passes a pointer directly -- no conversion or copy. This is why Win32 interop should prefer the `W` (wide) variants of API functions whenever possible.
```

**String as an output parameter:**

When native code writes a string into a caller-provided buffer, use `StringBuilder`:

```csharp
[DllImport("kernel32.dll", CharSet = CharSet.Unicode)]
static extern uint GetWindowsDirectory(StringBuilder lpBuffer, uint uSize);

// Usage:
var buffer = new StringBuilder(260); // MAX_PATH
GetWindowsDirectory(buffer, (uint)buffer.Capacity);
string winDir = buffer.ToString();
```

### Structs

Native APIs define structs with precise memory layouts. C# must match these layouts exactly, or the native code reads garbage. The `[StructLayout]` attribute gives you control.

**`LayoutKind.Sequential`** -- fields are laid out in declaration order, matching C struct behavior:

```csharp
[StructLayout(LayoutKind.Sequential)]
public struct RECT
{
    public int Left;
    public int Top;
    public int Right;
    public int Bottom;
}

[DllImport("user32.dll")]
static extern bool GetWindowRect(IntPtr hWnd, out RECT lpRect);
```

**`LayoutKind.Explicit`** -- you specify the exact byte offset of each field. This is how you create C-style **unions** where multiple fields share the same memory:

```csharp
[StructLayout(LayoutKind.Explicit)]
public struct InputUnion
{
    [FieldOffset(0)] public int IntValue;
    [FieldOffset(0)] public float FloatValue;  // Shares the same 4 bytes as IntValue
}
```

**Controlling struct packing:**

C compilers often pack structs with specific alignment. Use the `Pack` property to match:

```csharp
// Match a C struct compiled with #pragma pack(1)
[StructLayout(LayoutKind.Sequential, Pack = 1)]
public struct TightlyPacked
{
    public byte Flag;   // offset 0
    public int Value;   // offset 1 (not 4, because Pack = 1 disables padding)
}
```

```ad-warning
If your C# struct layout does not match the native struct's layout, you will get **silent data corruption** -- fields will read from the wrong byte offsets. This is one of the hardest bugs to diagnose in P/Invoke code. Always verify your layout against the native header file, paying close attention to packing, alignment, and pointer sizes (32-bit vs 64-bit).
```

### Arrays

**Fixed-size arrays within structs:**

Native structs often contain inline fixed-size arrays. Use `[MarshalAs]` with `UnmanagedType.ByValArray`:

```csharp
[StructLayout(LayoutKind.Sequential, CharSet = CharSet.Unicode)]
public struct WIN32_FIND_DATA
{
    public uint dwFileAttributes;
    public FILETIME ftCreationTime;
    public FILETIME ftLastAccessTime;
    public FILETIME ftLastWriteTime;
    public uint nFileSizeHigh;
    public uint nFileSizeLow;
    public uint dwReserved0;
    public uint dwReserved1;

    [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 260)]
    public string cFileName;

    [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 14)]
    public string cAlternateFileName;
}
```

**Passing managed arrays to native functions:**

```csharp
[DllImport("mylib.dll")]
static extern void ProcessData(
    [MarshalAs(UnmanagedType.LPArray, SizeParamIndex = 1)]
    int[] data,
    int length
);

// Usage:
int[] values = { 1, 2, 3, 4, 5 };
ProcessData(values, values.Length);
```

### Delegates as Function Pointers

When a native API requires a callback, you pass a C# delegate. The CLR creates a native-callable thunk that wraps your delegate.

```csharp
// Define a delegate type matching the native callback signature
public delegate bool EnumWindowsProc(IntPtr hWnd, IntPtr lParam);

[DllImport("user32.dll")]
static extern bool EnumWindows(EnumWindowsProc lpEnumFunc, IntPtr lParam);

// Usage:
EnumWindowsProc callback = (hWnd, lParam) =>
{
    Console.WriteLine($"Window: {hWnd}");
    return true; // continue enumeration
};

EnumWindows(callback, IntPtr.Zero);
```

```ad-danger
**Critical: prevent GC collection of your delegate.** When you pass a delegate to native code, the CLR gives the native side a function pointer to a thunk. But the garbage collector does not know that native code holds a reference to your delegate. If the delegate is only referenced by a local variable, the GC may collect it while native code still has the function pointer -- causing a **crash** when the native code tries to call back.

**Solution:** store the delegate in a field or static variable that outlives the native call:

~~~csharp
// BAD -- delegate may be collected during the native call
EnumWindows((hWnd, lParam) => { /* ... */ return true; }, IntPtr.Zero);

// GOOD -- delegate is rooted in a field
private static EnumWindowsProc _callback = (hWnd, lParam) => { /* ... */ return true; };
EnumWindows(_callback, IntPtr.Zero);
~~~
```

---

## The [MarshalAs] Attribute

The `[MarshalAs]` attribute lets you override the CLR's default marshalling behavior for a specific parameter, return value, or field. You specify which `UnmanagedType` the CLR should convert to/from.

### Common UnmanagedType Values

| UnmanagedType | C# Type | Native Type | Notes |
|---|---|---|---|
| `LPStr` | `string` | `char*` (ANSI) | CLR allocates temp buffer, converts UTF-16 to ANSI |
| `LPWStr` | `string` | `wchar_t*` (wide) | On Windows, may pin directly (same encoding) |
| `LPUTF8Str` | `string` | `char*` (UTF-8) | .NET 7+ -- useful for cross-platform native APIs |
| `Bool` | `bool` | `BOOL` (4 bytes) | Win32 convention: 0 = false, non-zero = true |
| `U1` | `bool` | `unsigned char` (1 byte) | C/C++ `bool` convention |
| `ByValTStr` | `string` | Inline char array | For fixed-size string fields in structs. Requires `SizeConst`. |
| `ByValArray` | Array | Inline array | For fixed-size arrays in structs. Requires `SizeConst`. |
| `LPArray` | Array | Pointer to array | For dynamically-sized arrays passed to functions |

### Examples

```csharp
// Override bool marshalling from default 4-byte BOOL to 1-byte
[DllImport("mylib.dll")]
static extern void SetFlag([MarshalAs(UnmanagedType.U1)] bool flag);

// Marshal string as UTF-8 instead of the default
[DllImport("mylib.dll")]
static extern void LogMessage([MarshalAs(UnmanagedType.LPUTF8Str)] string message);

// Marshal return value
[DllImport("mylib.dll")]
[return: MarshalAs(UnmanagedType.U1)]
static extern bool IsReady();
```

```ad-note
If you do not specify `[MarshalAs]`, the CLR uses **default marshalling rules** based on the C# type. For example, `string` defaults to `LPStr` (ANSI) for `CharSet.Ansi` and `LPWStr` (wide) for `CharSet.Unicode`. The defaults are usually fine for Win32 APIs, but cross-platform native libraries often expect UTF-8, which requires an explicit `[MarshalAs(UnmanagedType.LPUTF8Str)]`.
```

---

## Manual Marshalling with the Marshal Class

The `System.Runtime.InteropServices.Marshal` class provides methods for direct, manual control over unmanaged memory and data conversion. Use it when automatic marshalling is not flexible enough -- for example, when you receive a raw `IntPtr` from native code and need to interpret what it points to.

### Allocating and Freeing Unmanaged Memory

```csharp
// Allocate 100 bytes of unmanaged memory
IntPtr ptr = Marshal.AllocHGlobal(100);

try
{
    // Use the memory...
    // Native function writes data to ptr
    NativeFunction(ptr, 100);
}
finally
{
    // MUST free manually -- the GC does not track this memory
    Marshal.FreeHGlobal(ptr);
}
```

```ad-danger
Memory allocated with `Marshal.AllocHGlobal` (or `AllocCoTaskMem`) lives **outside the GC's awareness**. If you do not call the corresponding `Free` method, the memory leaks permanently -- no finalizer, no garbage collection, no safety net. Always pair allocation with deallocation in a `try/finally` block.
```

### Copying Between Managed and Unmanaged Memory

```csharp
byte[] managedArray = { 0x01, 0x02, 0x03, 0x04 };
IntPtr unmanagedPtr = Marshal.AllocHGlobal(managedArray.Length);

try
{
    // Managed → Unmanaged
    Marshal.Copy(managedArray, 0, unmanagedPtr, managedArray.Length);

    // Call native function that processes the data
    ProcessBuffer(unmanagedPtr, managedArray.Length);

    // Unmanaged → Managed (read results back)
    byte[] result = new byte[managedArray.Length];
    Marshal.Copy(unmanagedPtr, result, 0, result.Length);
}
finally
{
    Marshal.FreeHGlobal(unmanagedPtr);
}
```

### Reading Structs from Unmanaged Memory

When a native function returns a pointer to a struct, use `Marshal.PtrToStructure<T>` to read it:

```csharp
// Native function returns a pointer to a POINT struct
IntPtr pPoint = GetPointFromNative();

// Read the struct from unmanaged memory into a managed struct
POINT point = Marshal.PtrToStructure<POINT>(pPoint);
Console.WriteLine($"X: {point.X}, Y: {point.Y}");
```

### Writing Structs to Unmanaged Memory

```csharp
var rect = new RECT { Left = 0, Top = 0, Right = 100, Bottom = 100 };
IntPtr ptr = Marshal.AllocHGlobal(Marshal.SizeOf<RECT>());

try
{
    Marshal.StructureToPtr(rect, ptr, false);
    // Pass ptr to native function
    NativeSetRect(ptr);
}
finally
{
    Marshal.FreeHGlobal(ptr);
}
```

### String Conversion

```csharp
string message = "Hello, native world!";

// Convert to unmanaged ANSI string
IntPtr ansiPtr = Marshal.StringToHGlobalAnsi(message);
try
{
    NativeFunctionAnsi(ansiPtr);
}
finally
{
    Marshal.FreeHGlobal(ansiPtr);
}

// Convert to unmanaged Unicode (wide) string
IntPtr widePtr = Marshal.StringToHGlobalUni(message);
try
{
    NativeFunctionWide(widePtr);
}
finally
{
    Marshal.FreeHGlobal(widePtr);
}

// Read a string from an unmanaged pointer
IntPtr nativeStringPtr = GetNativeString();
string result = Marshal.PtrToStringUni(nativeStringPtr);   // wide
string resultAnsi = Marshal.PtrToStringAnsi(nativeStringPtr); // ANSI
```

### Marshal Class Quick Reference

| Method | Purpose |
|---|---|
| `AllocHGlobal(size)` | Allocate `size` bytes of unmanaged memory |
| `FreeHGlobal(ptr)` | Free memory allocated with `AllocHGlobal` |
| `AllocCoTaskMem(size)` | Allocate using the COM task allocator (use when native code will free with `CoTaskMemFree`) |
| `FreeCoTaskMem(ptr)` | Free memory allocated with `AllocCoTaskMem` |
| `Copy(src, dst, ...)` | Copy bytes between managed arrays and unmanaged pointers |
| `PtrToStructure<T>(ptr)` | Read a struct of type `T` from unmanaged memory |
| `StructureToPtr(struct, ptr, delete)` | Write a struct to unmanaged memory |
| `SizeOf<T>()` | Get the unmanaged size of a type (after marshalling) |
| `StringToHGlobalAnsi(s)` | Convert string to ANSI `char*` in unmanaged memory |
| `StringToHGlobalUni(s)` | Convert string to wide `wchar_t*` in unmanaged memory |
| `PtrToStringAnsi(ptr)` | Read ANSI string from unmanaged pointer |
| `PtrToStringUni(ptr)` | Read wide string from unmanaged pointer |

```ad-note
**`AllocHGlobal` vs `AllocCoTaskMem`** -- use `AllocHGlobal` when your code both allocates and frees the memory. Use `AllocCoTaskMem` when the memory will be freed by COM infrastructure or a native function that expects `CoTaskMemFree`. Mixing the two (allocating with one, freeing with the other) is undefined behavior.
```

---

## Performance Implications

The blittable vs non-blittable distinction has direct, measurable impact on P/Invoke performance.

### Blittable Path (Fast)

```
1. CLR pins the managed object (prevents GC from moving it)
2. CLR passes a raw pointer to the pinned memory
3. Native function reads/writes directly
4. CLR unpins the object
```

**Cost:** a single pin/unpin operation. No memory allocation, no copying, no conversion. This is essentially the same as a native function call with a pointer.

### Non-Blittable Path (Slow)

```
1. CLR allocates temporary unmanaged memory
2. CLR converts each non-blittable field from managed → native representation
3. CLR copies the converted data into the temporary buffer
4. CLR passes a pointer to the temporary buffer
5. Native function executes
6. CLR converts return/out values from native → managed
7. CLR copies the converted data back
8. CLR frees the temporary memory
```

**Cost:** memory allocation + conversion + two copies (in and out) + deallocation. For frequently-called functions (tight loops, real-time processing), this overhead is significant.

### Optimization Strategies

1. **Use blittable types wherever possible.** Replace `bool` with `int` (0/1) at the P/Invoke boundary. Use `IntPtr` instead of `string` and marshal strings manually only when needed.

2. **Prefer `CharSet.Unicode` for string parameters.** UTF-16 strings can be pinned directly on Windows, avoiding the ANSI conversion overhead.

3. **Use `in` parameters for large blittable structs.** The CLR passes a pointer to the pinned struct instead of copying it onto the stack:

```csharp
// The 'in' keyword tells the CLR to pass by reference
[DllImport("mylib.dll")]
static extern void ProcessLargeStruct(in LargeBlittableStruct data);
```

4. **Batch P/Invoke calls.** Each managed-to-native transition has overhead beyond marshalling (stack frame setup, GC pre-emption state changes). If you are calling a native function 10,000 times in a loop, consider redesigning the native API to accept a batch.

5. **Use `Span<T>` and `fixed` for zero-copy scenarios** in performance-critical code:

```csharp
unsafe
{
    int[] data = { 1, 2, 3, 4, 5 };
    fixed (int* ptr = data)
    {
        NativeProcessArray(ptr, data.Length);
    }
}
```

```ad-note
In modern .NET (5+), the runtime has further optimized the blittable marshalling path. For blittable structs smaller than a certain threshold, the JIT may even inline the entire P/Invoke call, eliminating the managed-to-native transition overhead entirely. This makes keeping your boundary types blittable even more rewarding.
```

---

## Summary Table

| Concept | What It Means |
|---|---|
| **Marshalling** | The process of converting data between managed (CLR) and unmanaged (native) representations when crossing the interop boundary |
| **Blittable** | A type with identical bit layout in managed and unmanaged memory -- can be passed by pointer with zero conversion (fast) |
| **Non-blittable** | A type that requires allocation, conversion, and copying to cross the boundary (slow) |
| **`[MarshalAs]`** | Attribute that overrides the default marshalling behavior for a parameter, field, or return value |
| **`[StructLayout]`** | Attribute that controls the memory layout of a struct (`Sequential`, `Explicit`, `Pack`) to match native struct definitions |
| **`Marshal` class** | Provides manual methods for allocating unmanaged memory, copying data, and converting types |
| **Pinning** | The CLR prevents the GC from moving an object so that a raw pointer to it remains valid during a native call |
| **`CharSet`** | Controls whether string parameters are marshalled as ANSI (`CharSet.Ansi`) or wide/UTF-16 (`CharSet.Unicode`) |
| **Thunk** | A native-callable wrapper generated by the CLR when a delegate is passed as a function pointer to native code |

---

## Related Notes

- [[Format of a .NET Assembly]] -- understanding PE headers and CLR metadata gives context for where managed types live
- [[The Role of .NET Assemblies]] -- assemblies as the unit of deployment, versioning, and type boundaries
- [[Satellite Assemblies]] -- localized resource assemblies also cross interop boundaries in certain scenarios
