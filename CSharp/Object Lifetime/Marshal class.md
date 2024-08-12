`Marshal` is a class in the `System.Runtime.InteropServices` namespace in .NET that provides a collection of methods for working with unmanaged code. It is used to perform various operations that involve interacting with unmanaged memory, such as allocating and deallocating unmanaged memory, copying data between managed and unmanaged memory, and converting managed types to unmanaged types and vice versa.

### Common Uses of `Marshal`

1. **Memory Allocation and Deallocation**:
   - `AllocHGlobal`: Allocates unmanaged memory.
   - `FreeHGlobal`: Frees unmanaged memory that was allocated with `AllocHGlobal`.

2. **Memory Copy**:
   - `Copy`: Copies data between managed and unmanaged memory.

3. **Pointers**:
   - `ReadInt32`, `ReadIntPtr`, `WriteInt32`, `WriteIntPtr`: Reads and writes values to unmanaged memory.

4. **Type Conversion**:
   - `StructureToPtr`: Converts a managed object to an unmanaged memory block.
   - `PtrToStructure`: Converts an unmanaged memory block to a managed object.
   - `SizeOf`: Returns the unmanaged size of a class or structure.

5. **String Conversion**:
   - `StringToHGlobalAnsi`, `StringToHGlobalUni`: Converts a managed string to an unmanaged string.
   - `PtrToStringAnsi`, `PtrToStringUni`: Converts an unmanaged string to a managed string.

### Example of `Marshal` Usage

Here’s a detailed example demonstrating some common operations with `Marshal`:

#### Example: Allocating and Deallocating Unmanaged Memory

```csharp
using System;
using System.Runtime.InteropServices;

public class MarshalExample
{
    public static void Main()
    {
        // Allocate unmanaged memory (100 bytes).
        IntPtr unmanagedPointer = Marshal.AllocHGlobal(100);

        try
        {
            // Initialize memory to zero.
            for (int i = 0; i < 100; i++)
            {
                Marshal.WriteByte(unmanagedPointer, i, 0);
            }

            // Write a value to the unmanaged memory.
            Marshal.WriteInt32(unmanagedPointer, 42);

            // Read the value back from the unmanaged memory.
            int value = Marshal.ReadInt32(unmanagedPointer);
            Console.WriteLine($"Value from unmanaged memory: {value}");
        }
        finally
        {
            // Free the unmanaged memory.
            Marshal.FreeHGlobal(unmanagedPointer);
        }
    }
}
```

#### Explanation:

1. **AllocHGlobal**:
   - `IntPtr unmanagedPointer = Marshal.AllocHGlobal(100);` allocates 100 bytes of unmanaged memory and returns a pointer to it.

2. **Write and Read Operations**:
   - `Marshal.WriteByte(unmanagedPointer, i, 0);` initializes each byte of the allocated memory to zero.
   - `Marshal.WriteInt32(unmanagedPointer, 42);` writes an integer value (42) to the unmanaged memory.
   - `int value = Marshal.ReadInt32(unmanagedPointer);` reads the integer value back from the unmanaged memory.

3. **FreeHGlobal**:
   - `Marshal.FreeHGlobal(unmanagedPointer);` frees the allocated unmanaged memory to avoid memory leaks.

### Example: Converting Structures

Here’s an example of converting a managed structure to unmanaged memory and back:

```csharp
using System;
using System.Runtime.InteropServices;

[StructLayout(LayoutKind.Sequential)]
public struct MyStruct
{
    public int Integer;
    public float FloatingPoint;
    [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 256)]
    public string Text;
}

public class MarshalStructExample
{
    public static void Main()
    {
        MyStruct myStruct = new MyStruct
        {
            Integer = 42,
            FloatingPoint = 3.14f,
            Text = "Hello, World!"
        };

        // Allocate unmanaged memory for the structure.
        IntPtr ptr = Marshal.AllocHGlobal(Marshal.SizeOf(myStruct));

        try
        {
            // Copy the structure to unmanaged memory.
            Marshal.StructureToPtr(myStruct, ptr, false);

            // Read the structure back from unmanaged memory.
            MyStruct anotherStruct = Marshal.PtrToStructure<MyStruct>(ptr);
            Console.WriteLine($"Integer: {anotherStruct.Integer}");
            Console.WriteLine($"FloatingPoint: {anotherStruct.FloatingPoint}");
            Console.WriteLine($"Text: {anotherStruct.Text}");
        }
        finally
        {
            // Free the unmanaged memory.
            Marshal.FreeHGlobal(ptr);
        }
    }
}
```

#### Explanation:

1. **StructLayout and MarshalAs**:
   - `[StructLayout(LayoutKind.Sequential)]` ensures the fields are laid out sequentially in memory.
   - `[MarshalAs(UnmanagedType.ByValTStr, SizeConst = 256)]` specifies how the string should be marshaled to unmanaged memory.

2. **SizeOf**:
   - `Marshal.SizeOf(myStruct)` gets the size of the structure in unmanaged memory.

3. **StructureToPtr**:
   - `Marshal.StructureToPtr(myStruct, ptr, false);` copies the structure to unmanaged memory.

4. **PtrToStructure**:
   - `Marshal.PtrToStructure<MyStruct>(ptr)` converts the unmanaged memory back to a managed structure.

5. **FreeHGlobal**:
   - `Marshal.FreeHGlobal(ptr);` frees the allocated unmanaged memory.

The `Marshal` class is a powerful tool for working with unmanaged code and resources, allowing you to handle memory and data conversions between managed and unmanaged contexts effectively.